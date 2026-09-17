# Compliance and Regulatory Boundaries

`xsdcompat` is a general-purpose developer tool for XSD-level change and compatibility analysis. This document defines what the project may and may not claim in regulated or compliance-sensitive environments.

## No certification claim

The project must not claim that use of `xsdcompat` makes a system, message, submission, organization, or migration compliant with any law, regulation, framework, or industry standard.

In particular, the project does not certify or attest compliance with ISO 20022, NIEM, FDA submission requirements, HIPAA, PCI DSS, SOX, 21 CFR Part 11, FedRAMP, NIST controls, or similar regimes.

The tool may analyze XSD artifacts used within those environments. That is a narrower claim.

## XSD is not the complete contract

A successful XSD compatibility result does not establish end-to-end compatibility. Real standards and enterprise integrations may additionally depend on:

- business rules;
- Schematron;
- external code lists and terminology;
- usage/implementation guidelines;
- generated bindings;
- transformations and mappings;
- application behavior;
- database constraints;
- transport/protocol rules;
- organization-specific policy.

Reports and documentation must preserve this boundary.

## Safe wording

Acceptable wording includes:

> No breaking changes were found under the selected XSD compatibility rules.

> This report evaluates XSD-level compatibility only.

Avoid wording such as:

> The migration is safe.

> This schema is ISO 20022 compliant.

> This release satisfies FDA requirements.

## Secure development frameworks

The project intends to align its engineering process with broadly applicable secure-development practices such as:

- NIST Secure Software Development Framework (SSDF);
- OpenSSF secure-development guidance;
- relevant AWS Well-Architected Security, Reliability, and Operational Excellence principles;
- Go's published security guidance.

Alignment is a development objective, not a certification claim.

## Regulated user data

Schemas may themselves reveal proprietary data models, field names, code values, patterns, or business concepts. Treat schemas and generated reports according to the user's own data-classification policy.

The runtime therefore has no telemetry, no update check, and no remote schema resolution. Hosted CI upload of reports is a user-controlled action outside the core tool.

## Standards fixtures

Third-party standards material used as test data must follow `docs/FIXTURE_POLICY.md`. The repository must not redistribute proprietary standards or customer schemas without documented rights.

## EU Cyber Resilience Act

Open-source legal obligations may depend on how the project is distributed, monetized, or stewarded. The project should reassess applicable Cyber Resilience Act obligations before introducing commercial distribution, paid binaries, bundled commercial offerings, or sustained commercial stewardship.

This repository does not provide legal advice. Material changes to commercialization or distribution should receive appropriate legal review.

## Changes requiring review

Update this document when:

- product marketing changes materially;
- standards-specific functionality is added;
- commercial distribution begins;
- the tool begins processing data beyond local XSD/catalog inputs;
- a certification, regulated use case, or formal assurance is proposed.
