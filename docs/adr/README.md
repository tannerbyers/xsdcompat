# Architecture Decision Records

Architecture Decision Records (ADRs) capture durable decisions whose context and tradeoffs would otherwise be easy to lose.

## Status values

- **Proposed** — under active review; implementation should not assume it is permanent.
- **Accepted** — current architectural direction.
- **Superseded** — replaced by a later ADR.
- **Deprecated** — retained for history but no longer recommended.

## Initial ADRs

- [ADR-0001: Go and zero third-party product dependencies](0001-go-zero-dependencies.md)
- [ADR-0002: Offline-only runtime](0002-offline-runtime.md)
- [ADR-0003: Conservative compatibility classification](0003-conservative-classification.md)
- [ADR-0004: XSD 1.0 semantic-analysis scope](0004-xsd-scope.md)

## When an ADR is required

Use an ADR for changes such as:

- adding a production dependency;
- enabling runtime network access;
- changing compatibility semantics;
- expanding XSD-version guarantees;
- adding hosted/runtime-plugin infrastructure;
- changing a major security/trust boundary;
- deliberately breaking a frozen 1.0 public contract.

Do not create ADRs for routine implementation details that can be safely changed without altering architectural constraints.

A new ADR should supersede rather than rewrite historical accepted decisions when the reasoning materially changes.
