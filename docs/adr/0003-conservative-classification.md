# ADR-0003: Conservative compatibility classification

**Status:** Accepted  
**Date:** 2026-09-17

## Context

XSD compatibility is not equivalent to text diffing. Some changes have straightforward implications, while others require deeper language/value-space reasoning. Returning `NON_BREAKING` merely because no known breaking rule matched would create false confidence and could cause unsafe CI automation.

## Decision

The semantic result model is:

- `BREAKING` — supported rules prove incompatibility;
- `NON_BREAKING` — supported rules prove compatibility;
- `REVIEW` — a meaningful change exists but the engine cannot prove either result;
- `ERROR` — a trustworthy schema graph/result could not be produced.

The engine must fail toward `REVIEW`/`ERROR`, never toward optimistic compatibility.

Overall result precedence after successful compilation is:

```text
BREAKING > REVIEW > NON_BREAKING
```

Policy/waivers are separate from semantic classification. A waived breaking change remains `BREAKING` and may receive a policy disposition such as `WAIVED`.

## Consequences

### Positive

- avoids representing absence of knowledge as proof;
- safer for automated enterprise CI gating;
- permits incremental rule coverage without requiring a full XSD theorem prover;
- makes unsupported XSD semantics visible.

### Negative

- early versions may produce more manual-review results;
- users may initially prefer a binary pass/fail result and require education on the distinction;
- rule implementations require documented proof boundaries and tests.

## Examples

Straightforward cardinality/facet transformations may be classifiable automatically.

Arbitrary regex changes, complex particle restructuring, XSD 1.1 assertions, or other unmodeled semantics normally produce `REVIEW` unless a specific rule later proves the relationship.

## Rule requirement

Any rule that emits `BREAKING` or `NON_BREAKING` must have documented semantic reasoning and tests covering the boundary where the reasoning stops applying.
