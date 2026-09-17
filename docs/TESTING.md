# Testing and Conformance Strategy

`xsdcompat` is correctness-sensitive infrastructure. The test strategy is designed to detect false compatibility classifications, schema-resolution defects, nondeterminism, resource-exhaustion bugs, and accidental expansion of the trusted computing base.

## Test layers

### Unit tests

Every compatibility rule must include focused tests covering:

- backward compatibility;
- forward compatibility;
- full compatibility;
- safe boundary cases;
- breaking boundary cases;
- the nearest ambiguous/unsupported case that must produce `REVIEW`.

Parser/compiler bug fixes require a permanent regression fixture.

### Semantic invariance tests

These changes should produce no semantic diff when effective behavior is unchanged:

- namespace-prefix renames;
- formatting/whitespace changes;
- XML comments;
- equivalent explicit versus implicit defaults;
- declaration movement across included files;
- `xs:choice` member reordering;
- `xs:all` member reordering.

`xs:sequence` order remains semantically significant.

### Schema-composition tests

Required coverage includes:

- standalone schemas;
- `xs:include`;
- nested includes;
- `xs:import`;
- nested imports;
- import/include cycles;
- duplicate sources;
- duplicate namespaces;
- chameleon includes;
- one chameleon source under multiple effective namespaces;
- `xml:base` where supported;
- XML Catalog mappings and catalog cycles.

### QName and namespace tests

Cover QName-valued attributes including:

- `type`;
- `base`;
- `ref`;
- `itemType`;
- `memberTypes`;
- `substitutionGroup`.

Include default namespaces, prefix shadowing, unbound prefixes, imported namespaces, local/global declarations, and no-namespace schemas.

### Type/content-model tests

Cover:

- named and anonymous simple/complex types;
- `simpleContent` / `complexContent`;
- extension/restriction;
- sequence/choice/all;
- model-group and attribute-group references;
- occurrence constraints;
- wildcards;
- mixed content;
- list and union types;
- substitution groups;
- abstract components;
- `nillable`;
- `block` / `final`;
- form defaults;
- supported facets;
- default/fixed values;
- identity-constraint syntax capture.

### Unsupported-feature tests

Unsupported constructs must never silently disappear. Tests must verify documented behavior for at least:

- `xs:redefine`;
- XSD 1.1 `xs:assert`;
- XSD 1.1 alternatives/open content as applicable;
- unknown XSD constructs;
- unmapped remote schema locations.

If an unsupported construct prevents trustworthy schema construction, the expected result is `ERROR`. If it can be represented but compatibility cannot be proven, the expected result is `REVIEW`.

## Adversarial and resource-limit tests

Test below, at, and above configured limits for:

- total source bytes;
- individual source size;
- source count;
- XML depth;
- schema graph depth;
- component count;
- derivation traversal;
- substitution closure;
- diagnostics/result count;
- catalog recursion.

Include recursive types, recursive groups, large enumerations, malformed XML, invalid QNames, path traversal, symlink escape attempts, and intentionally cyclic catalogs/imports.

## Fuzzing

Maintain Go fuzz targets for high-risk boundaries:

- XML token/namespace scanner;
- QName parser/resolver;
- URI/reference resolver;
- XML Catalog parser;
- XSD syntax parser;
- schema compiler.

A fuzz-discovered panic or incorrect invariant becomes a permanent regression test.

## Determinism tests

Golden text/JSON/Markdown/SARIF outputs must not depend on:

- Go map iteration order;
- wall-clock time;
- absolute checkout path;
- random identifiers;
- host-specific directory separators where normalized output is expected.

Run repeated analyses and compare serialized outputs byte-for-byte where appropriate.

## Race tests

Run `go test -race ./...` in CI once code exists. The public library should be safe for independent concurrent calls even if the implementation itself remains primarily single-threaded.

## Representative corpora

Use representative public corpora where licensing permits, with provenance recorded per `docs/FIXTURE_POLICY.md`.

Priority corpora:

- NIEM schema sets for substitution groups, imports, wildcards, lists/unions, catalogs, and large component graphs;
- ISO 20022 messages for mainstream enterprise XSD constructs and realistic financial schemas;
- additional public edge-case schemas when they expose a specific compiler behavior.

Do not include proprietary ACORD/customer schemas without redistribution rights.

## Differential/oracle testing

Development or scheduled CI may compare selected behavior against independent XSD processors such as Java `SchemaFactory`/Xerces, Helium, `jacoelho/xsd`, or Python `xmlschema`.

These implementations are test oracles, not runtime dependencies or automatic authorities. A disagreement must be resolved against the XSD specification and converted into a focused regression fixture.

## Compatibility-rule evidence

A rule may emit `BREAKING` or `NON_BREAKING` only when its semantic reasoning is documented in `docs/RULES.md` and exercised by tests. If the rule depends on semantics the compiler does not model, the expected classification is `REVIEW`.

## CI tiers

Normal pull requests should eventually run:

```text
gofmt check
go vet ./...
go test ./...
go test -race ./...
zero-product-dependency invariant
fixture provenance checks
golden determinism tests
```

Scheduled/security jobs may add:

```text
govulncheck ./...
extended fuzzing
large real-schema corpus
differential/oracle tests
cross-platform build smoke tests
```

Release candidates run all applicable checks plus release-artifact verification.

## Pre-1.0 conformance gate

Before 1.0, the project must demonstrate that its semantic compiler can load and normalize the representative supported corpora with documented unsupported constructs and without optimistic fallback behavior.

Passing an external validator's test suite is useful evidence but does not by itself establish `xsdcompat` compatibility correctness, because this project intentionally implements a narrower semantic-analysis surface than a full XSD validator.
