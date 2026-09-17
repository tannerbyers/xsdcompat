# xsdcompat

`xsdcompat` is a planned zero-dependency Go CLI and library for deterministic XSD schema-set change and compatibility analysis.

> **Status:** design / pre-implementation. There is no stable release yet.

The project is intended to answer a bounded question well: **what changed between two XSD schema sets, and which changes are provably breaking, provably non-breaking, or require human review?**

## Design goals

- Single standalone Go binary.
- Zero third-party Go modules in the shipped product.
- Offline by architecture: no telemetry, update checks, or remote schema fetching.
- Deterministic output for the same tool version, ruleset, configuration, and inputs.
- Conservative compatibility analysis: uncertain semantics become `REVIEW`, never an optimistic pass.
- Useful in local development, CI, regulated-enterprise migration reviews, and other tooling through a stable machine-readable report.
- XSD 1.0 semantic analysis as the primary compatibility target.

## Non-goals

`xsdcompat` is **not** intended to be:

- a general XML instance validator;
- an XSD code generator;
- a WSDL/SOAP toolkit;
- a schema registry;
- a Schematron or XPath engine;
- an XSD 1.1 assertion solver;
- a hosted service;
- a replacement for business rules, code lists, usage guidelines, or application-level regression testing.

A successful XSD-level result does **not** mean an end-to-end integration or regulated migration is safe.

## Compatibility results

After a schema set is successfully compiled, findings use three semantic classifications:

- `BREAKING` — a supported rule proves the change is incompatible under the selected direction.
- `NON_BREAKING` — a supported rule proves the change is compatible under the selected direction.
- `REVIEW` — the tool detected a meaningful change but cannot safely prove compatibility.

Input, resolution, or compiler failures produce `ERROR` rather than a compatibility result.

The core rule is:

```text
if provably breaking:
    BREAKING
else if provably non-breaking:
    NON_BREAKING
else:
    REVIEW
```

Policy and waivers never rewrite semantic truth. A waived breaking change remains `BREAKING`; policy may only mark it as intentionally accepted for enforcement purposes.

## Planned interfaces

The portable product contract will be the CLI plus a versioned JSON report. A small Go API will also be provided.

Planned CLI shape:

```text
xsdcompat diff OLD NEW
xsdcompat check OLD NEW
xsdcompat inspect SCHEMA
xsdcompat rules
xsdcompat explain RULE-ID
xsdcompat version
```

Planned output formats:

- terminal text;
- JSON;
- Markdown;
- SARIF.

## Architecture

The implementation will compile XSD source into an immutable semantic component graph before diffing. It will not compare source text or XML DOM trees directly.

```text
schema files
    |
    v
secure local source layer
    |
    v
namespace-aware XSD syntax parser
    |
    v
schema graph compiler
(import/include/chameleon refs/types/groups/substitution)
    |
    v
immutable semantic IR
    |
    +---- old ----+
    |             |
    +---- new ----+
          |
          v
     semantic diff
          |
          v
 compatibility rules
          |
          v
 text / JSON / Markdown / SARIF
```

See [DESIGN.md](DESIGN.md) for the detailed architecture and constraints.

## Security model

The planned runtime intentionally has a small attack surface:

- local files only;
- no runtime network resolver;
- no external entity processing;
- workspace-bounded filesystem access;
- explicit resource/work limits for untrusted schema input;
- no plugin runtime;
- no shell hooks;
- no telemetry.

See [SECURITY.md](SECURITY.md) and [docs/THREAT_MODEL.md](docs/THREAT_MODEL.md).

## Dependency policy

The shipped Go module is expected to contain **no third-party Go modules**. External development, CI, vulnerability-scanning, SBOM, or release tools may be used without entering the product dependency graph.

A dependency may only be proposed when implementing the required capability ourselves would present a demonstrably greater correctness or security risk. See [DESIGN.md](DESIGN.md#dependency-exception-policy).

## Standards scope

The compatibility engine targets XSD 1.0 semantics first. XSD 1.1 constructs should never be silently ignored; where they can be represented but not safely analyzed they will produce `REVIEW`, and where they prevent construction of a trustworthy schema graph they will produce `ERROR`.

The test corpus is expected to include representative open standards such as NIEM and ISO 20022 where licensing permits. Third-party fixtures require explicit provenance and license review. See [docs/FIXTURE_POLICY.md](docs/FIXTURE_POLICY.md).

## Documentation

- [Technical design](DESIGN.md)
- [Compatibility contract](docs/COMPATIBILITY.md)
- [Threat model](docs/THREAT_MODEL.md)
- [Release and supply-chain policy](docs/RELEASE.md)
- [Fixture and IP policy](docs/FIXTURE_POLICY.md)
- [Architecture decisions](docs/adr/README.md)
- [Contributing](CONTRIBUTING.md)
- [Security reporting](SECURITY.md)
- [Support policy](SUPPORT.md)
- [Governance](GOVERNANCE.md)

## License

Apache License 2.0. See [LICENSE](LICENSE).

Contributions are expected to use Developer Certificate of Origin (`Signed-off-by`) sign-off as described in [CONTRIBUTING.md](CONTRIBUTING.md).
