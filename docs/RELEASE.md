# Release and Supply-Chain Policy

This document defines the intended release process for `xsdcompat`. The project is pre-release; exact automation may evolve, but the security properties below should remain stable.

## Release goals

Official releases should be:

- built from a tagged repository revision;
- produced by repository-controlled CI, not a maintainer laptop;
- reproducible as far as practical;
- traceable to source commit and workflow;
- checksummed;
- accompanied by an SBOM and build provenance by 1.0;
- built with a currently security-supported Go toolchain;
- free of third-party Go modules unless an approved ADR changes the dependency policy.

## Versioning

The project intends to use Semantic Versioning.

Before `1.0.0`, public behavior may change between minor releases. The repository should still document meaningful compatibility changes clearly.

At 1.0, the following become explicit compatibility contracts:

- public Go API;
- CLI arguments and command semantics;
- configuration schema;
- JSON report schema;
- stable rule IDs;
- exit behavior;
- documented compatibility definitions.

The rule engine also has a `rulesetVersion` because a corrected classification may change CI outcomes without changing the CLI/API shape.

## Supported Go versions

Release builds should use the current patched Go toolchain selected by the project and remain within Go's active security-support window.

CI may test more than one Go version when useful, but the release toolchain should be pinned explicitly rather than implicitly following `latest`.

## Release artifacts

Planned 1.0 artifacts:

```text
xsdcompat_<version>_linux_amd64
xsdcompat_<version>_linux_arm64
xsdcompat_<version>_darwin_amd64
xsdcompat_<version>_darwin_arm64
xsdcompat_<version>_windows_amd64.exe
checksums.txt
SPDX SBOM
build provenance / artifact attestation
release notes
source archive
```

Additional architectures should be added only when there is demonstrated use and the release/test matrix can support them responsibly.

## Reproducibility

Production binaries must not intentionally embed:

- current wall-clock build time;
- builder username;
- absolute checkout path;
- random build identifiers;
- mutable environment-dependent values.

`xsdcompat version` may expose deterministic build metadata such as:

- semantic version;
- source commit;
- ruleset version;
- Go version.

Do not claim fully reproducible builds until the release process verifies that independent builds of the same revision/toolchain/configuration produce matching artifacts or documents any unavoidable differences.

## CI permission model

GitHub Actions workflows should use least privilege.

Requirements when workflows are added:

- default token permissions should be read-only;
- jobs receive only permissions they require;
- release permissions are isolated from ordinary PR jobs;
- untrusted fork/PR code never receives release credentials;
- OIDC/short-lived credentials are preferred over long-lived secrets;
- third-party actions are pinned by immutable commit SHA rather than mutable tags;
- environments/approval gates may be used for release publishing where appropriate.

## Core PR checks

Expected baseline checks:

```text
gofmt verification
go vet ./...
go test ./...
go test -race ./...
zero-product-dependency invariant
fixture provenance/license checks
golden report determinism
```

Security/extended checks may include:

```text
govulncheck ./...
fuzz/corpus testing
large real-schema corpus
cross-platform build smoke tests
oracle/differential validation jobs
```

## Zero-dependency invariant

The shipped module should satisfy the documented zero-third-party-module decision.

CI should fail if the module graph contains an unexpected module. A future production dependency requires an ADR and associated license/security review before this check is changed.

External CI tools do not violate this constraint as long as they do not enter the distributed Go module/binary dependency graph.

## Vulnerability scanning

`govulncheck` or the current official Go vulnerability tooling should run in security/release automation.

A Go standard-library/runtime vulnerability that reaches `xsdcompat` should be handled like any other relevant product vulnerability even though there are no third-party Go modules.

## SBOM

Publish an SPDX SBOM by 1.0.

The zero-dependency architecture should make the software inventory intentionally small, but the SBOM remains useful evidence for:

- package/project identity;
- version;
- source revision;
- license;
- toolchain/build metadata.

## Checksums

Every binary release should publish a SHA-256 manifest. The checksum manifest itself is not a substitute for provenance/attestation, but provides a convenient integrity check.

## Provenance and artifact attestation

By 1.0, release automation should publish verifiable provenance showing that official artifacts were built from the expected repository, commit/tag, and controlled workflow.

GitHub Artifact Attestations are the preferred initial mechanism if the repository/account capabilities support them.

## Release pipeline

Target flow:

```text
merge reviewed change
      |
      v
required CI passes
      |
      v
tag release
      |
      v
clean cross-platform builds
      |
      v
binary smoke tests
      |
      v
govulncheck / security gates
      |
      v
reproducibility check
      |
      v
checksums + SBOM + provenance
      |
      v
publish GitHub release
```

Official release binaries must not be built locally and manually attached to a release.

## Release notes

Release notes should explicitly call out:

- breaking public API/CLI/config/report changes;
- rules whose classifications changed;
- newly supported/unsupported XSD constructs;
- security fixes;
- changes to resource limits;
- Go minimum/toolchain support changes;
- dependency-policy changes;
- known compatibility-analysis limitations.

## Patch releases

Correctness bugs that can produce false compatibility results should be considered for patch releases even when they are not traditional security vulnerabilities.

In particular, a false `NON_BREAKING` classification should receive high priority because downstream automation may trust it.

## Release keys and secrets

The project should avoid long-lived personal signing/publishing credentials where platform-native OIDC or attestations can provide traceability instead.

If signing keys are introduced later, key custody, rotation, revocation, and maintainer-transition procedures must be documented before use.
