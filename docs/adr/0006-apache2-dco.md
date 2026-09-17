# ADR-0006: Apache-2.0 Licensing with DCO Contributions

**Status:** Accepted

## Context

The project is intended for adoption inside commercial and regulated organizations. The licensing model should be permissive, easy to review, and explicit about contributor patent rights while keeping contribution administration lightweight.

## Decision

License the project under Apache License 2.0. Contributions use Developer Certificate of Origin 1.1 sign-off rather than a bespoke CLA initially.

Third-party material must retain required attribution and provenance. Proprietary/customer schemas must not be committed without documented redistribution rights.

## Consequences

- Commercial and internal use are permitted under a familiar permissive license.
- Apache-2.0 provides an explicit patent grant from contributors.
- DCO keeps contribution overhead low while requiring contributors to certify their right to submit the work.
- Relicensing or additional commercial rights are not assumed; a future need would require a separate decision.

## Rejected alternatives

- MIT: acceptable but lacks Apache-2.0's explicit patent grant.
- GPL/AGPL: creates avoidable enterprise adoption/licensing friction for this project goal.
- Custom license: increases legal review and ambiguity.
- CLA from day one: unnecessary without a concrete need for rights beyond Apache-2.0.
