# ADR-0010: Compatibility Ruleset Versioning

**Status:** Accepted

## Context

A compatibility-classification fix can change CI outcomes even when the CLI, JSON schema, and Go API remain source-compatible. Treating rule behavior as invisible implementation detail would make upgrades surprising for regulated/controlled workflows.

## Decision

Expose a distinct `rulesetVersion` in machine-readable reports and version documentation. Stable rule IDs are never reused for different semantics.

Release notes must identify rule additions or classification changes that can alter outcomes. Before 1.0 the ruleset may evolve quickly; at 1.0 its versioning behavior becomes part of the public contract.

## Consequences

- Consumers can attribute historical findings to the exact semantic rules used.
- Compatibility fixes can be communicated without pretending they are ordinary formatting changes.
- CI users can decide when to adopt classification-changing releases.
- Golden tests must cover ruleset-dependent output deterministically.

## Rejected alternatives

- Tool version only: insufficiently precise for rules-engine behavior.
- Rule text as identity: unstable and unsuitable for automation.
- Reusing rule IDs after semantic rewrites: breaks historical reports.
