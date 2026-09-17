# Threat Model

`xsdcompat` processes potentially untrusted XML Schema and catalog files. This document defines the security boundary and required mitigations.

## Assets to protect

- host filesystem outside the configured workspace;
- confidential schema content;
- CI credentials and release credentials;
- host/network resources;
- process availability and memory;
- trustworthiness of compatibility results;
- integrity of published release artifacts.

## Trust boundaries

### Untrusted input

Treat the following as attacker-controlled unless the caller explicitly knows otherwise:

- XSD bytes;
- file names/relative paths supplied by schema references;
- XML namespace declarations and QName values;
- XML Catalog files and catalog mappings;
- deeply nested/recursive schema graphs;
- unusually large numeric/cardinality values;
- annotations and foreign metadata;
- configuration supplied by a CI/project repository.

### Trusted code boundary

The shipped executable consists of project code plus the supported Go toolchain/standard library. Product code must not depend on third-party Go modules unless an approved ADR supersedes that constraint.

## Threats and required controls

### 1. External entity expansion / XXE

**Threat:** input causes the parser to read local files or external resources through entities/DTDs.

**Controls:**

- do not process external general or parameter entities;
- reject DTD use in XSD documents;
- catalog DOCTYPE declarations, where tolerated for interoperability, must never cause DTD retrieval or expansion;
- test XXE/local-file payloads explicitly.

### 2. SSRF / unintended network access

**Threat:** `schemaLocation`, catalog entries, or another URI causes outbound HTTP/DNS traffic.

**Controls:**

- runtime has no remote schema resolver;
- `http:` and `https:` identifiers require explicit local mapping or fail;
- no telemetry or update checking;
- no implicit DNS/network probes.

### 3. Filesystem traversal

**Threat:** relative paths, symlinks, catalog mappings, or `xml:base` escape the intended workspace and read host files.

**Controls:**

- CLI opens an explicit root and performs traversal-resistant rooted access;
- absolute host paths outside the root are rejected;
- test `..`, symlink, encoded, and platform-specific escape cases;
- library API favors explicit caller-provided `fs.FS`/source abstractions.

### 4. Recursive graph denial of service

**Threat:** cyclic includes/imports/types/groups/substitution relationships cause infinite recursion, stack overflow, or repeated work.

**Controls:**

- explicit graph states (`unseen`, `resolving`, `resolved`, `failed`);
- memoized component resolution;
- bounded dependency/derivation/substitution work;
- prefer iterative traversal for adversarial-depth paths;
- cycle-specific tests.

### 5. Memory exhaustion

**Threat:** very large files, declaration counts, enumerations, diagnostics, or graph fan-out consume unbounded memory.

**Controls:**

- per-source and aggregate byte limits;
- source/component/diagnostic count limits;
- bounded retained source metadata;
- avoid generic DOM retention where unnecessary;
- benchmark peak memory on representative large schema sets.

### 6. CPU / algorithmic denial of service

**Threat:** crafted model groups, substitution closure, derivation graphs, or repeated reference resolution create pathological work.

**Controls:**

- explicit cumulative work budgets for operations whose cost depends on schema shape;
- avoid re-traversing resolved graphs;
- complexity-aware tests and benchmarks;
- stop with a structured limit error rather than continuing indefinitely.

### 7. XML nesting / parser stack exhaustion

**Threat:** deeply nested XML or recursive internal walkers overflow the stack.

**Controls:**

- explicit XML/schema-depth limits;
- no uncontrolled recursive traversal over attacker-shaped structures;
- fuzz and synthetic deep-nesting tests.

### 8. Namespace/QName confusion

**Threat:** incorrect namespace resolution causes the analyzer to bind references to the wrong type/element, potentially returning a false compatibility result.

**Controls:**

- retain an explicit namespace context at each QName-bearing XSD location;
- test imported references, default namespaces, prefix rebinding, no-namespace schemas, and chameleon includes;
- component identity uses expanded names, not lexical prefixes;
- ambiguous/unresolved references are errors, not best guesses.

### 9. Chameleon/source-identity confusion

**Threat:** the same physical no-namespace schema included under different target namespaces is incorrectly cached as one semantic document.

**Controls:**

- distinguish physical `SourceID` from semantic `SchemaViewID`;
- include effective namespace in schema-view identity;
- regression tests using the same source under multiple namespaces.

### 10. Malicious XML Catalogs

**Threat:** catalog recursion, redirects, or local mappings escape security boundaries or consume resources.

**Controls:**

- implement only documented catalog directives;
- bounded `nextCatalog` recursion;
- catalog targets remain local/root-bounded;
- unsupported directives produce explicit diagnostics when relevant;
- remote identifiers are never dereferenced automatically.

### 11. Diagnostic/output leakage

**Threat:** output exposes host absolute paths, source contents, sensitive environment values, or excessive proprietary contract information.

**Controls:**

- normal output uses logical/relative source identifiers;
- do not print source contents implicitly;
- no environment dump;
- debug mode must be explicit;
- document that reports themselves can contain schema component names, patterns, enumerations, and other potentially confidential metadata.

### 12. False `NON_BREAKING` result

**Threat:** automation trusts a result the engine did not actually prove.

**Controls:**

- strict `BREAKING` / `NON_BREAKING` / `REVIEW` model;
- unknown semantics propagate to `REVIEW` or `ERROR`;
- no heuristic rename or probabilistic compatibility decisions;
- rule-level proof rationale and boundary tests;
- semantic classification cannot be overridden by policy.

A false `NON_BREAKING` caused by unsupported semantics should be treated as a high-severity correctness issue because downstream CI may use it as an enforcement signal.

### 13. Panic / internal invariant exposure

**Threat:** malformed input triggers panic, crash loops, stack traces, or path leakage.

**Controls:**

- user-controlled invalid input returns structured errors;
- fuzzing at scanner/parser/resolution/compiler boundaries;
- top-level CLI may convert unexpected internal panic to a generic internal diagnostic while avoiding sensitive stack output unless explicit debug mode is enabled;
- every discovered crash gets a permanent regression test.

### 14. Configuration abuse

**Threat:** configuration becomes an execution surface.

**Controls:**

- JSON/declarative configuration only initially;
- no embedded scripting;
- no command hooks;
- no environment-variable interpolation;
- no runtime plugins;
- versioned config schema with unknown-field behavior explicitly tested/documented.

### 15. Build/release compromise

**Threat:** attacker publishes a modified binary or compromises CI credentials.

**Controls:**

- protected release path;
- minimal GitHub Actions permissions;
- pin third-party Actions by immutable commit SHA when introduced;
- OIDC/short-lived credentials where possible;
- official artifacts built only by repository-controlled workflow;
- SHA-256 manifest;
- SBOM;
- provenance/artifact attestations;
- no manually uploaded locally built official binary.

### 16. Go runtime/stdlib vulnerabilities

**Threat:** zero third-party dependencies creates false confidence while the Go toolchain itself has a relevant vulnerability.

**Controls:**

- use currently security-supported Go versions;
- run `govulncheck`/official equivalent in CI/release checks;
- monitor Go security releases;
- rebuild/patch affected release artifacts when warranted.

## Resource-limit design

Exact limits must be measured against representative corpora before 1.0. Each limit must have tests immediately below, at, and above its boundary.

Potential controls include:

- maximum source files;
- maximum bytes per source;
- maximum aggregate schema bytes;
- maximum XML nesting depth;
- maximum schema graph expansion depth;
- maximum declarations/components;
- maximum attributes/particles per component where justified;
- maximum derivation/substitution traversal work;
- maximum catalog hops;
- maximum findings/diagnostics.

The project should prefer cumulative work budgets when a simple depth/count limit does not bound algorithmic work adequately.

## Out of scope

The project does not attempt to protect callers who deliberately hand report files or schema content to external systems. For example, uploading SARIF to a hosted CI provider is the caller's explicit data-sharing decision, not runtime behavior by `xsdcompat`.

## Security testing expectations

Security-focused tests should include at minimum:

- XXE/entity payloads;
- DTD handling;
- attempted remote schema retrieval;
- path and symlink traversal;
- catalog cycles and escapes;
- deep nesting;
- recursive includes/types/groups;
- high fan-out substitution and model graphs;
- malformed namespace/QName input;
- huge/invalid occurrence values;
- panic/fuzz corpus regression cases;
- deterministic handling of limit exhaustion.
