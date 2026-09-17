# Compatibility Rule Registry

This document defines how compatibility rules are identified, documented, tested, and versioned. The concrete rule list will grow with implementation.

## Rule ID format

Stable IDs should use:

```text
XSD-<SUBJECT>-<CHANGE>
```

Examples:

```text
XSD-ELEMENT-ADDED
XSD-ELEMENT-REMOVED
XSD-MIN-OCCURS-INCREASED
XSD-MAX-OCCURS-DECREASED
XSD-ENUM-VALUE-REMOVED
XSD-PATTERN-CHANGED
XSD-SUBSTITUTION-MEMBER-REMOVED
```

Once released as stable, an ID must not be reused to mean a different semantic change.

## Required rule metadata

Each implemented rule should have a single source-of-truth definition containing at least:

- stable rule ID;
- short title;
- affected semantic component/change type;
- supported compatibility directions;
- possible classifications;
- human-readable explanation;
- standards/semantic rationale;
- test fixture references where practical.

Human documentation and `xsdcompat rules` / `xsdcompat explain` output should eventually be generated from, or directly consume, the same registry used by the rule engine so they cannot drift.

## Classification bar

A rule may emit `BREAKING` or `NON_BREAKING` only when the engine can prove that result from represented XSD semantics.

Rules must not use:

- filename similarity;
- lexical prefix identity;
- undocumented generator/runtime behavior;
- guessed application semantics;
- probabilistic confidence;
- heuristic rename matching

as evidence for semantic compatibility.

If the engine cannot prove the result, use `REVIEW`.

## Candidate initial rule families

The following are design candidates, not yet implementation commitments.

### Elements and attributes

- added/removed declaration;
- required/optional transition;
- `minOccurs` increase/decrease;
- `maxOccurs` increase/decrease;
- nillable transition;
- abstract transition;
- effective namespace/form change;
- default/fixed change where value semantics are safely understood.

### Simple types

- enumeration member added/removed;
- min/max length widened/narrowed;
- exact length change;
- supported numeric bound widened/narrowed;
- total/fraction digit changes;
- list/union composition change;
- pattern change (normally `REVIEW` until a specific safe rule exists).

### Complex types/content models

- base-type/derivation changes;
- particle additions/removals;
- supported sequence/choice/all transformations;
- group/attribute-group effective change;
- wildcard policy change;
- mixed/simple/element-only content transition.

### Substitution and derivation controls

- substitution-group membership added/removed;
- `block`/`final` effective changes;
- base-type chain changes.

### Identity constraints

- key/unique/keyref added/removed/changed, with conservative classification when XPath semantics are not evaluated.

### Unsupported/XSD 1.1 semantics

- assertion change;
- conditional type alternative change;
- open-content change;
- other detected but unsupported semantics.

These should default to `REVIEW` or `ERROR` according to whether a trustworthy XSD 1.0 semantic graph can still be built.

## Directional tests

Every rule that classifies compatibility must have tests for each direction it claims to understand:

```text
backward
forward
full
```

At minimum, tests should include:

- positive/proven case;
- opposite-direction behavior;
- boundary value;
- unsupported/ambiguous nearby case;
- imported/referenced variant where reference resolution could affect the result.

## Ruleset version

The machine-readable report will contain `rulesetVersion` separately from `toolVersion`.

A ruleset-version policy will be frozen before 1.0. Its purpose is to let CI/audit consumers identify the exact semantic decision set used to produce a historical report.

Changes that correct a false classification must be prominently documented even if they do not require a major CLI/API version bump.

## Policy interaction

Rules classify semantic truth. Policy determines what to do with findings.

A policy may:

- fail on `REVIEW`;
- allow `REVIEW`;
- waive a particular `BREAKING` finding;
- attach an issue/ticket/reason/expiry.

Policy must not modify the rule's semantic classification.
