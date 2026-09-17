# xsdcompat Technical Design

**Status:** proposed implementation baseline  
**Primary language:** Go  
**License:** Apache-2.0  
**Product dependency target:** Go standard library only  
**Primary semantic target:** XSD 1.0 schema-set compatibility analysis

This document records the intended architecture, boundaries, security posture, and major implementation constraints for `xsdcompat`. Major changes to these decisions should be accompanied by an ADR under `docs/adr/`.

## 1. Problem statement

Teams migrating XSD-based contracts need to determine what structurally changed, where a component is used, whether previously valid XML remains valid, whether newly valid XML remains acceptable to the old schema, and which changes require manual investigation.

Text/XML diffs are insufficient because XSD semantics depend on namespaces, references, type derivation, particles, imports/includes, substitution groups, defaults, wildcards, and restrictions.

`xsdcompat` therefore compares **compiled semantic schema models**, not source text.

## 2. Product guarantee

The tool may state that a supported rule proves a change breaking or non-breaking under a defined XSD compatibility direction. It must not claim that an application, business process, regulatory submission, or overall migration is safe.

A schema may be only one contract layer. Business rules, Schematron, external code sets, usage guidelines, generated bindings, transformations, and application behavior are outside the XSD-level guarantee unless a future feature explicitly and independently supports them.

## 3. Core principles

### 3.1 Correctness before coverage

Unknown or unsupported compatibility semantics produce `REVIEW` or `ERROR`, never an optimistic pass.

```text
if provably breaking:
    BREAKING
else if provably non-breaking:
    NON_BREAKING
else:
    REVIEW
```

### 3.2 Zero product dependencies

The shipped Go module and binary should contain no third-party Go modules. This reduces supply-chain surface, license review, vulnerability triage, and long-term upgrade work.

External CI, fuzzing, vulnerability, SBOM, provenance, or oracle tooling may be used without becoming product dependencies.

### 3.3 Offline by architecture

Runtime schema analysis performs no network access. There is no telemetry, update check, account, remote schema resolver, or hosted component.

Remote identifiers may be mapped explicitly to local resources through supported XML Catalog rules, but the runtime never dereferences them over the network.

### 3.4 Determinism

The same tool version, ruleset, configuration, and input bytes must produce equivalent reports.

Normal reports must not depend on:

- Go map iteration order;
- current wall-clock time;
- host absolute paths;
- random IDs;
- environment-specific ordering.

### 3.5 Bounded work

Every operation derived from untrusted schema input must have a finite limit or a demonstrably bounded algorithm. Limits will cover source bytes, file count, XML depth, schema graph expansion, components, derivation walks, substitution closure, catalog recursion, diagnostics, and other potentially explosive work.

Defaults should be calibrated from representative real corpora rather than guessed before implementation.

### 3.6 Immutable compiled state

Schema compilation publishes an immutable `SchemaSet`. Diff and compatibility analysis read it but do not mutate it.

## 4. Goals

Version 1.0 should provide:

- standalone cross-platform executable;
- small embeddable Go API;
- XSD 1.0 schema-set parsing and semantic compilation;
- local `xs:include` and `xs:import` resolution;
- chameleon include handling;
- namespace/QName resolution;
- a documented useful subset of OASIS XML Catalog URI resolution;
- deterministic semantic component graph;
- backward, forward, and full compatibility modes;
- `BREAKING`, `NON_BREAKING`, and `REVIEW` classifications;
- structured operational errors;
- source file/line provenance;
- text, JSON, Markdown, and SARIF reporting;
- policy/waiver support that does not alter semantic classification;
- runtime with no network activity;
- signed/attested releases, checksums, and SBOM by 1.0.

## 5. Non-goals

The 1.x architecture intentionally excludes:

- general XML instance validation;
- XSD code generation;
- WSDL/SOAP processing;
- Schematron execution;
- XPath evaluation;
- XSD regex execution;
- arbitrary content-language inclusion proofs;
- automatic schema/application migration;
- generated-client compatibility guarantees;
- schema registry functionality;
- SaaS/hosted operation;
- runtime plugins;
- AI-generated compatibility decisions;
- implicit remote schema fetching.

These exclusions are maintenance and correctness controls, not missing implementation tasks.

## 6. Standards scope

### 6.1 XSD 1.0

XSD 1.0 is the primary semantic compatibility target.

The compiler must model the component relationships required for the target enterprise schema sets, including:

- global/local elements and attributes;
- named and anonymous simple/complex types;
- `sequence`, `choice`, and `all`;
- model groups and attribute groups;
- simple/complex content;
- extension and restriction relationships;
- `minOccurs`/`maxOccurs`;
- imports/includes and chameleon includes;
- substitution groups;
- abstract/nillable/block/final semantics;
- element/attribute form defaults;
- wildcards and `anyAttribute`;
- enumerations and supported length/numeric facets;
- lists and unions;
- default/fixed values;
- identity-constraint syntax/provenance.

### 6.2 XSD 1.1

XSD 1.1 constructs must not disappear silently.

Where an XSD 1.1 construct can be captured without compromising the XSD 1.0 model, affected changes may be reported as `REVIEW`. Where a construct changes schema construction in an unsupported way, compilation must fail explicitly.

Examples:

- `xs:assert` — capture/detect; compatibility normally `REVIEW`;
- `xs:alternative` — capture/detect; compatibility normally `REVIEW`;
- open content — initially `REVIEW` unless later modeled safely;
- schema composition features that invalidate the compiler model — `ERROR`.

### 6.3 `xs:redefine`

`xs:redefine` is out of initial scope because ignoring it would construct an incorrect component graph and correct implementation has pervasive replacement semantics.

Encountering an unsupported `xs:redefine` must result in an explicit compiler error, not a partial pass.

## 7. Architecture

```text
SchemaSetSpec
   |
   v
secure local source layer
   |
   v
namespace-aware XML/XSD scanner
   |
   v
typed XSD syntax representation
   |
   v
schema graph compiler
(include/import/chameleon/symbols/refs/derivation/groups/substitution)
   |
   v
immutable semantic IR
   |
   +-------- old --------+
   |                     |
   +-------- new --------+
             |
             v
        semantic diff
             |
             v
      compatibility rules
             |
             v
       policy/enforcement
             |
      text/JSON/MD/SARIF
```

## 8. Package structure

Initial implementation should keep almost all implementation packages internal:

```text
cmd/xsdcompat/
internal/source/
internal/xmlscan/
internal/catalog/
internal/xsdsyntax/
internal/schema/
internal/diff/
internal/compat/
internal/policy/
internal/report/
xsdcompat.go
```

Only a deliberately small root API should become public before 1.0.

## 9. Proposed public API

Conceptually:

```go
type SchemaSetSpec struct {
    FS       fs.FS
    Entries  []string
    Catalogs []string
}

type Options struct {
    Compatibility CompatibilityMode
    Limits        Limits
    Policy        Policy
}

func Compare(
    ctx context.Context,
    oldSet SchemaSetSpec,
    newSet SchemaSetSpec,
    opts Options,
) (*Report, error)
```

Requirements:

- callers supply filesystem/source context explicitly;
- no implicit process-global filesystem or network access in the library;
- `context.Context` is accepted for cancellation and never stored;
- no package-level mutable state;
- public types must not expose internal parser representation.

## 10. Secure source layer

CLI filesystem access must be confined to an explicit workspace root using a traversal-resistant API (`os.Root` where supported by the chosen minimum Go version, or an equivalently proven design).

Reject or safely resolve:

- absolute paths outside the workspace;
- `..` traversal escaping the workspace;
- symlink escapes;
- unmapped `http:`/`https:` schema locations;
- external entities;
- DTD use in XSD input;
- shell/environment expansion.

The library should prefer caller-provided `fs.FS`/source abstractions rather than taking arbitrary paths itself.

## 11. XML and namespace parsing

`encoding/xml` is used as a tokenizer, not as the semantic schema representation.

The scanner must retain an explicit namespace context at the point where QName-valued attributes occur. Important QName-valued attributes include `type`, `base`, `ref`, `itemType`, `memberTypes`, and `substitutionGroup`.

```go
type QName struct {
    Namespace string
    Local     string
}
```

The scanner should retain source identity and position information needed for diagnostics. It should not retain a generic DOM-like tree once typed XSD syntax has been admitted unless a measured need justifies it.

## 12. Schema source identity and chameleon includes

A physical source file is not sufficient semantic identity because one no-namespace schema can be chameleon-included under different target namespaces.

Use separate identities:

```go
type SourceID string

type SchemaViewID struct {
    Source             SourceID
    EffectiveNamespace string
}
```

The compiler must permit the same raw source to participate in multiple semantic views.

## 13. Compilation stages

Schema compilation should follow explicit, testable stages:

1. Acquire explicit entry schemas.
2. Tokenize source securely.
3. Capture typed namespace-aware XSD syntax.
4. Discover include/import edges.
5. Resolve local/catalog references.
6. Close the schema source graph.
7. Establish semantic schema views.
8. Register global symbols.
9. Resolve QName references.
10. Resolve model groups and attribute groups.
11. Resolve type derivation.
12. Resolve substitution groups.
13. Compute effective defaults.
14. Build semantic content models.
15. Validate internal compiler invariants.
16. Publish immutable `SchemaSet`.

Reference resolution must use explicit state such as `unseen`, `resolving`, `resolved`, and `failed`; recursive XSD graphs must not be handled through uncontrolled recursion.

## 14. XML Catalogs

Catalog support is intended for deterministic local mapping, not remote discovery.

Initial catalog support should be a documented subset demonstrated by real target fixtures, likely including `uri`, `rewriteURI`, and `nextCatalog`. Unsupported directives must produce diagnostics rather than being silently ignored where they may affect resolution.

Catalog documents may contain a historical catalog DOCTYPE, but the tool must not retrieve or process remote DTDs.

## 15. Semantic IR

The immutable IR should represent effective semantics plus declaration provenance. Important concepts include:

- `SchemaSet`;
- `ComponentID`;
- `QName`;
- `SourceLocation`;
- element/attribute declarations;
- simple/complex type definitions;
- particles/model groups;
- wildcards;
- attribute groups;
- substitution relationships;
- identity-constraint syntax;
- supported facet sets.

Global identity is normally component kind plus QName. Local identities need stable deterministic ownership paths, never Go map order or transient compiler indexes.

## 16. Semantic normalization

The diff layer should eliminate representation-only noise.

Examples expected to be semantically unchanged when effective behavior is unchanged:

- namespace prefix changes;
- formatting/comments;
- explicit declaration of an already-effective default;
- movement between included source files;
- source-order changes where XSD semantics are unordered.

Order must remain meaningful for `xs:sequence`.

## 17. Compatibility model

Backward compatibility means instances valid under the old schema remain valid under the new schema.

Conceptually:

```text
L(old) ⊆ L(new)
```

Forward compatibility reverses the relation. Full compatibility requires both directions.

The engine must not attempt arbitrary language-inclusion proofs. Each rule proves a bounded semantic implication.

## 18. Rule architecture

Rules have stable IDs. IDs are never reused for different semantics.

A finding should include at minimum:

- rule ID;
- semantic classification;
- compatibility direction;
- stable component identity;
- old/new source locations where applicable;
- before/after values;
- human-readable reasoning;
- dependency/impact path where available.

Only rules with defensible semantics may return `BREAKING` or `NON_BREAKING`. Complex or unsupported transformations return `REVIEW`.

## 19. Value-space policy

Do not build a complete XSD datatype validator merely to compare facets.

Implement only value semantics required to prove selected compatibility rules, using standard-library facilities such as `math/big` where appropriate. Datatypes whose semantic equality/ordering is not implemented safely must fall back to conservative handling.

Pattern expressions are stored and compared, but arbitrary XSD-regex language inclusion is not evaluated in 1.x.

Identity-constraint XPath is retained as syntax/provenance but not executed.

## 20. Unsupported-feature propagation

Unsupported content has three categories:

1. **Opaque and outside XSD validity semantics** — e.g. foreign application metadata in `xs:annotation/xs:appinfo`; ignored for XSD compatibility with scope clearly documented.
2. **Representable but compatibility unknown** — affected findings become `REVIEW`.
3. **Prevents trustworthy schema construction** — compilation produces `ERROR` and no overall compatibility verdict.

## 21. Result precedence

After successful compilation:

```text
if any BREAKING:
    overall = BREAKING
else if any REVIEW:
    overall = REVIEW
else:
    overall = NON_BREAKING
```

Compilation/resolution failure produces `ERROR` rather than a semantic result.

## 22. Policy and waivers

Policy controls enforcement, not semantic classification.

A waived incompatibility must remain represented as:

```text
classification: BREAKING
disposition: WAIVED
```

not `NON_BREAKING`.

Waivers should be attributable and bounded where possible (reason, issue/ticket reference, optional expiry).

## 23. Configuration

Use JSON initially because it is available in the standard library and avoids introducing a YAML dependency.

Configuration is versioned and deliberately declarative. It must not include executable hooks, environment interpolation, embedded scripts, or plugins.

## 24. CLI and exit behavior

Planned commands:

```text
xsdcompat diff OLD NEW
xsdcompat check OLD NEW
xsdcompat inspect SCHEMA
xsdcompat rules
xsdcompat explain RULE-ID
xsdcompat version
```

`diff` reports semantic results without treating `BREAKING` as an operational failure. `check` applies enforcement policy and returns a non-zero process status when the configured policy blocks the change. Operational/configuration/compiler failure must be distinguishable from a compatibility-policy failure.

Exact exit codes will be frozen before 1.0 and documented as a public contract.

## 25. Machine-readable report

The JSON report should separately version:

```text
formatVersion
configVersion
rulesetVersion
toolVersion
```

Reports should include deterministic SHA-256 fingerprints of the logical old/new source sets and effective configuration. Fingerprints must be generated from a canonical sorted manifest, not host-specific paths or timestamps.

## 26. Threat model summary

Primary threats include:

- XXE/external entity retrieval;
- SSRF through schema locations;
- path/symlink traversal;
- recursive include/type graph denial of service;
- oversized schema inputs;
- deep XML nesting;
- diagnostic/result memory explosion;
- namespace confusion;
- catalog cycles/malicious mappings;
- algorithmic blowups;
- parser panics;
- build/release supply-chain tampering.

Controls are detailed in `docs/THREAT_MODEL.md`.

## 27. Go coding expectations

Production code should satisfy:

- `gofmt` and `go vet` clean;
- no panic for user-controlled invalid input;
- no package-level mutable state;
- no nondeterministic map output;
- no uncontrolled recursion;
- no unbounded input-derived allocation;
- long-running loops check context cancellation where appropriate;
- errors retain causes (`%w`) where useful;
- libraries return structured diagnostics and do not log;
- concurrency is introduced only when measured need justifies the complexity.

## 28. Testing strategy

Required suites include:

- unit tests for each rule in backward/forward/full directions;
- semantic invariance tests;
- compiler composition tests (`include`, `import`, cycles, chameleon views, catalogs);
- namespace/QName edge cases;
- groups, derivation, substitution, wildcards, lists/unions;
- adversarial limit tests;
- native Go fuzzing for scanner/resolution/parser/compiler boundaries;
- race tests;
- deterministic golden reports;
- representative open standards corpora where licensing permits.

Separate development jobs may compare selected fixtures with independent XSD processors. Those processors are oracles for investigation, not production dependencies or unquestioned authorities.

## 29. Third-party fixture policy

All third-party test data requires recorded provenance and licensing. Proprietary schemas must not be committed without explicit redistribution permission. See `docs/FIXTURE_POLICY.md`.

## 30. Supply-chain and releases

By 1.0, official releases should be produced only by repository-owned CI and include:

- cross-platform binaries;
- SHA-256 checksum manifest;
- SPDX SBOM;
- build provenance/attestation;
- release notes;
- source archive.

Builds should avoid embedding timestamps, usernames, checkout paths, or other host-specific data. See `docs/RELEASE.md`.

## 31. Licensing and contributions

Project code is Apache-2.0. Contributions use DCO sign-off. Third-party source code should not be copied into the core implementation without explicit license/provenance review and an architecture decision when it affects the zero-dependency policy.

## 32. Regulatory positioning

`xsdcompat` is a general-purpose developer tool. It must not claim certification or regulatory compliance with standards or regimes such as ISO 20022, NIEM, FDA submission requirements, HIPAA, PCI DSS, SOX, FedRAMP, or 21 CFR Part 11.

It can be used as one technical control or migration aid inside processes governed by those standards.

## 33. Dependency exception policy

Adding a production dependency requires an ADR demonstrating all of the following:

1. the capability is required for correctness or a committed product requirement;
2. implementing it internally creates greater security/correctness risk;
3. the dependency is actively maintained;
4. its license is acceptable;
5. its full transitive graph and release provenance are understood;
6. relevant security history has been reviewed;
7. dependency-specific types do not leak into the public API;
8. a credible replacement/removal path exists.

Convenience alone is insufficient. CLI parsing, colors, JSON, logging, YAML preference, table formatting, and test assertion helpers do not meet the bar by themselves.

## 34. Implementation phases

### Phase 0 — repository baseline

License, CI skeleton, security/support/contribution policies, ADRs, and package skeleton.

### Phase 1 — semantic compiler feasibility

Secure source layer, namespace/QName handling, typed XSD syntax, composition graph, symbols/references, type/group/substitution resolution.

**Gate:** representative NIEM and ISO 20022 schema sets compile into correct semantic graphs using only the Go standard library.

### Phase 2 — semantic normalization

Immutable deterministic `SchemaSet`, effective defaults/derivation/models/facets/wildcards/provenance.

### Phase 3 — semantic diff

Describe structural changes without compatibility classification. Representation-only changes must disappear.

### Phase 4 — compatibility rules

Add high-confidence rules incrementally with directional tests and explicit review boundaries.

### Phase 5 — policy and reporting

JSON/text/Markdown/SARIF, rule explanations, waivers, enforcement behavior.

### Phase 6 — security hardening

Resource-limit calibration, fuzzing, traversal tests, malicious corpus, vulnerability checks, determinism/reproducibility work.

### Phase 7 — real migration corpus validation

Use legally distributable historical schema pairs and private/local corpora to measure misses, false findings, review frequency, unsupported constructs, memory, and runtime.

### Phase 8 — 1.0

Freeze public Go API, CLI, report/config formats, rule IDs/ruleset, exit behavior, support/security/release policies.

## 35. Zero-dependency go/no-go gate

The zero-dependency design is validated when Phase 1 demonstrates that representative target schema sets can be compiled correctly without implementing validator-only subsystems such as XPath execution, XSD regex execution, or XML instance validation.

Reconsider the architecture only if required real schema sets cannot be modeled correctly, the semantic compiler unexpectedly requires validator-level machinery, or differential/corpus testing exposes systemic correctness problems.

If a dependency becomes necessary, evaluate one coherent, mature semantic XSD implementation behind a strict adapter rather than accumulating unrelated utility dependencies.

## 36. Definition of 1.0 readiness

1.0 is not ready until:

- there are no third-party Go modules unless the dependency ADR has explicitly superseded this baseline;
- representative target schema sets compile;
- imports/includes/chameleon behavior has regression coverage;
- unsupported semantics never silently disappear;
- input-controlled recursive work is bounded;
- malformed input cannot routinely panic the tool under corpus/fuzz testing;
- semantic normalization removes representation noise;
- every breaking/non-breaking rule has documented reasoning and tests;
- uncertain inference becomes `REVIEW`;
- all output formats agree semantically;
- config/report/ruleset versions are explicit;
- official binaries are checksummed, SBOM'd, and attested;
- security, contribution, support, licensing, fixture, and governance policies are published;
- documentation does not imply regulatory certification or end-to-end migration safety.
