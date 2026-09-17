# Governance

`xsdcompat` currently uses a lightweight maintainer-led governance model appropriate for an early-stage open-source project.

## Maintainers

Maintainers are responsible for:

- project scope and architecture;
- release decisions;
- compatibility-rule semantics;
- security response;
- reviewing contributions;
- licensing and fixture provenance;
- maintaining public API and ruleset compatibility commitments.

The initial maintainer is the repository owner. Additional maintainers may be added when there is sustained contribution and demonstrated familiarity with the project's correctness and security constraints.

## Decision making

Routine implementation decisions are made through pull-request review.

Changes to durable architectural constraints should be recorded as Architecture Decision Records under `docs/adr/`. Examples include:

- adding a production dependency;
- enabling runtime network access;
- changing the semantic compatibility model;
- broadening XSD-version guarantees;
- introducing plugins or hosted components;
- breaking the stable machine-readable report contract after 1.0.

ADRs document context, decision, consequences, and superseded decisions. They are not immutable; replacing one requires an explicit superseding ADR.

## Compatibility decisions

A `BREAKING` or `NON_BREAKING` rule is part of the project's trusted semantic behavior. Rule changes should be evidence-driven and reviewed independently from unrelated changes.

When evidence is insufficient, the default classification is `REVIEW`.

## Releases

Only maintainers publish official releases. Release artifacts should be built by repository-controlled automation as documented in [docs/RELEASE.md](docs/RELEASE.md).

## Contributions

Contributions follow [CONTRIBUTING.md](CONTRIBUTING.md) and require DCO sign-off.

Maintainers may decline contributions that are technically valid but significantly expand maintenance scope, attack surface, dependency burden, or project responsibilities beyond the documented design.

## Conduct

Project interactions should remain professional, technical, and respectful. Harassment, threats, discriminatory abuse, doxxing, or deliberately disruptive behavior are not acceptable in project spaces.

If community scale warrants a more formal code of conduct and enforcement process, one can be adopted explicitly rather than adding a policy the project is not yet equipped to administer consistently.
