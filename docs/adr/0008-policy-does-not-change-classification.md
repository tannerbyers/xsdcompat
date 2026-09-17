# ADR-0008: Policy Does Not Rewrite Semantic Classification

**Status:** Accepted

## Context

Enterprise workflows need waivers and policy exceptions for intentional breaking changes. If policy can rewrite the semantic result itself, reports cease to be trustworthy historical evidence.

## Decision

Semantic classification and enforcement disposition are separate fields.

A finding that is semantically `BREAKING` remains `BREAKING` even when policy marks it `WAIVED` or allows the build to proceed. Configuration may control enforcement severity, expiry, justification, and CI behavior, but must not transform semantic truth.

## Consequences

- Audit reports preserve what the engine actually concluded.
- Teams can accept intentional incompatibilities without hiding them.
- Report schemas need separate classification and disposition concepts.
- Expired/invalid waivers can change enforcement without recalculating semantics.

## Rejected alternatives

- Configuration overrides `BREAKING` to `NON_BREAKING`: destroys auditability.
- Suppress waived findings entirely: hides meaningful migration evidence.
