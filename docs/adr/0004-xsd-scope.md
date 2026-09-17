# ADR-0004: XSD 1.0 semantic-analysis scope

**Status:** Accepted  
**Date:** 2026-09-17

## Context

The target regulated/enterprise schema ecosystems require real XSD semantics: namespaces/QNames, imports/includes, chameleon schemas, type derivation, groups, substitution groups, wildcards, lists/unions, and occurrence/facet rules. However, implementing a complete XSD validator would also require substantial validator-only machinery such as full value-space validation, XSD regex execution, identity-constraint XPath execution, and XML instance assessment.

XSD 1.1 adds additional complexity including assertions, conditional type alternatives, and open content.

## Decision

Version 1.x will target XSD 1.0 **semantic schema analysis**, not general XML instance validation.

The compiler will model the XSD 1.0 component relationships required for trustworthy diff/compatibility analysis.

XSD 1.1 constructs must never be silently ignored:

- where they can be represented without invalidating the component graph, affected changes normally become `REVIEW`;
- where they prevent trustworthy component construction, compilation returns `ERROR`.

`xs:redefine` is initially unsupported and produces an explicit compiler error because silently ignoring/reducing its component-replacement semantics would produce an incorrect graph.

## Non-goals preserved by this decision

- arbitrary XML instance validation;
- XSD regex evaluation;
- XPath evaluation;
- key/keyref runtime execution;
- universal content-model language inclusion;
- XSD 1.1 assertion solving.

## Consequences

### Positive

- keeps the implementation materially smaller than a full XSD validator;
- permits zero-dependency Go while still supporting real enterprise schema sets;
- focuses testing on the semantic component compiler and compatibility rules;
- avoids claiming standards conformance beyond the actual product purpose.

### Negative

- some XSD 1.1 or uncommon XSD 1.0 constructs will require manual review or error;
- users may need a separate validator for XML instance testing;
- project documentation must clearly distinguish schema analysis from validation/certification.

## Validation gate

The design remains viable only if representative target corpora (including NIEM and ISO 20022 examples where licensing permits) can be compiled correctly without introducing validator-only subsystems.

If that gate fails systemically, the architecture should be reconsidered through a new ADR rather than progressively reimplementing an entire validator by accident.
