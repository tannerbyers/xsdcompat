# ADR-0009: No Runtime Plugin System

**Status:** Accepted

## Context

A plugin system could allow custom compatibility rules or resolvers, but it would create a long-lived compatibility surface and expand the security, licensing, distribution, and debugging burden of a deliberately small offline tool.

## Decision

Do not provide a runtime plugin system in the 1.x architecture.

Customization is limited to declarative configuration, policy, waivers, explicit local sources/catalog mappings, and stable machine-readable output that external tooling can consume.

## Consequences

- The shipped binary remains a closed, auditable execution unit.
- Rule behavior remains tied to a known ruleset version.
- Organizations needing custom logic can wrap the CLI/JSON contract instead of loading arbitrary code into the process.
- New core rules require normal project review and release rather than private plugins.

## Rejected alternatives

- Go `plugin`: platform limitations and ABI/versioning burden.
- Embedded scripting: introduces interpreter/security and determinism concerns.
- Dynamically loaded third-party rule packages: conflicts with zero-dependency and supply-chain goals.
