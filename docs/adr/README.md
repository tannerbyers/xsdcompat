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
- [ADR-0005: CLI and versioned JSON as the portable public contract](0005-cli-json-public-contract.md)
- [ADR-0006: Apache-2.0 licensing with DCO contributions](0006-apache2-dco.md)
- [ADR-0007: Workspace-bounded filesystem access](0007-secure-workspace-filesystem.md)
- [ADR-0008: Policy does not rewrite semantic classification](0008-policy-does-not-change-classification.md)
- [ADR-0009: No runtime plugin system](0009-no-runtime-plugin-system.md)
- [ADR-0010: Compatibility ruleset versioning](0010-ruleset-versioning.md)

## When an ADR is required

Use an ADR for changes such as:

- adding a production dependency;
- enabling runtime network access;
- changing compatibility semantics;
- expanding XSD-version guarantees;
- adding hosted/runtime-plugin infrastructure;
- changing licensing/contribution terms;
- changing a major security/trust boundary;
- changing the portable CLI/JSON contract materially;
- allowing policy to alter semantic classification;
- changing ruleset identity/versioning behavior;
- deliberately breaking a frozen 1.0 public contract.

Do not create ADRs for routine implementation details that can be safely changed without altering architectural constraints.

A new ADR should supersede rather than rewrite historical accepted decisions when the reasoning materially changes.
