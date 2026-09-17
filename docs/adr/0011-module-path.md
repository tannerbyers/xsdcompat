# ADR-0011: Permanent Go module path

- **Status:** Accepted
- **Date:** 2026-09-17

## Context

The embeddable Go API makes the module path part of the public source-level contract. Moving the repository to another owner or organization after consumers import the package would require downstream import-path changes. Go major versions after v1 also use semantic import versioning (`/v2`, `/v3`, and so on).

The project owner does not intend to create a separate GitHub organization for `xsdcompat`.

## Decision

The canonical module path is permanently:

```text
github.com/tannerbyers/xsdcompat
```

The initial `go.mod` MUST use that module path.

If the project ever publishes an incompatible v2 or later Go API, it will follow Go semantic import versioning, for example:

```text
github.com/tannerbyers/xsdcompat/v2
```

Repository transfers that would change the canonical module path are considered breaking changes and require a superseding ADR before they occur.

## Consequences

- Consumers can rely on the personal GitHub namespace as the canonical import path.
- No organization migration is planned or required before implementation.
- Future incompatible major versions must follow Go's major-version module path rules.
- Documentation, examples, package comments, and release metadata should consistently use the canonical path.