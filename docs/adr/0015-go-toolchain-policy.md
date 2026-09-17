# ADR-0015: Go toolchain and module-graph policy

- **Status:** Accepted
- **Date:** 2026-09-17

## Context

`xsdcompat` is intended for long-lived enterprise use with a minimal audit and supply-chain surface. Go's standard library/toolchain is still an upstream dependency and has its own support/security lifecycle. The root module must also remain free of accidental tool or library dependencies.

## Decision

When implementation begins:

- `go.mod` uses the canonical module path from ADR-0011;
- the `go` directive names the oldest Go release the project intentionally supports;
- CI tests the oldest supported release and the newest currently supported Go release where practical;
- official binaries are built with a current security-patched supported Go toolchain pinned by release automation;
- the root `go.mod` contains no third-party `require`, `replace`, or `tool` directives unless an approved ADR supersedes the zero-dependency decision;
- external CI/release/security utilities remain outside the product module graph;
- a `toolchain` directive is not added merely for developer convenience because it can trigger toolchain acquisition behavior and complicate offline enterprise workflows;
- Go security releases are reviewed and affected binaries are rebuilt when warranted.

The exact supported Go versions may advance over time without superseding this ADR as long as the policy above remains unchanged.

## Consequences

- Consumers get a conventional Go module without third-party dependency resolution.
- Security review can distinguish the product module graph from external repository automation.
- CI must include a module-graph invariant check.
- Changes to the supported Go window must be documented in release notes and user documentation.