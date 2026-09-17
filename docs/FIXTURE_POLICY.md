# Test Fixture and Intellectual-Property Policy

Real schema corpora are important for correctness, but standards and customer schemas can carry redistribution, attribution, trademark, confidentiality, or contractual restrictions. This policy applies before any third-party XSD/XML/catalog fixture is committed to the public repository.

## Default rule

Prefer small synthetic fixtures authored specifically for `xsdcompat`.

Third-party fixtures are added only when they provide coverage that synthetic fixtures cannot reasonably reproduce or when they are useful as a representative integration corpus.

## Required provenance

Each third-party fixture set must record:

- source organization/project;
- canonical source location;
- version/release/date or source commit when available;
- license/terms of use;
- required attribution;
- files included;
- whether files were modified/minimized;
- reason for inclusion;
- reviewer who verified redistribution terms.

Recommended layout:

```text
testdata/third_party/<fixture>/
    SOURCE.json
    LICENSE.txt or NOTICE.txt (when applicable)
    ...fixture files...
```

`SOURCE.json` should be machine-readable so CI can later verify that every third-party fixture has provenance metadata.

## Synthetic fixtures

Synthetic fixtures authored for the project are Apache-2.0 project content unless documented otherwise.

When minimizing a third-party schema to create a reproduction, do not automatically assume the result is newly authored/free of the source license. If substantial third-party expression remains, retain the required provenance/license or recreate the scenario independently.

## Proprietary/customer schemas

Do not commit:

- customer schemas;
- employer-internal schemas;
- licensed proprietary standards;
- schemas received under NDA;
- production data samples;
- regulated/confidential instance documents;

unless explicit public redistribution rights have been verified and documented.

A schema being downloadable or usable by an employee/customer is not evidence of redistribution permission.

Sensitive schemas may be used locally to reproduce bugs, but they should be replaced with sanitized/minimal synthetic fixtures before a public fix is merged whenever possible.

## ISO 20022

ISO 20022 publishes repository information under its documented IPR/terms framework. Any ISO-derived fixture included here must retain clear provenance and must not be represented as the official, complete, or current ISO 20022 source.

Before committing a fixture, verify the current terms at the canonical ISO 20022 site and record the exact source/release used.

Do not use ISO trademarks/logos in a way that implies endorsement.

## NIEM / OASIS

NIEM/OASIS materials may have different licenses for specifications, model content, code, and other artifacts. Retain the license/attribution that applies to the exact files used.

Current NIEM model/specification artifacts should be treated according to their repository and OASIS license notices, not assumed to inherit the `xsdcompat` Apache-2.0 license.

## FDA / U.S. government materials

Many U.S. federal-government materials are public domain, but exceptions can apply to third-party content, trademarks, logos, or incorporated material.

Check the exact FDA/resource-page policy for the artifact before vendoring it and record provenance. Do not use FDA logos or wording that implies FDA endorsement/certification of this project.

## ACORD and other membership/proprietary standards

Do not vendor ACORD or similar proprietary/member-only schemas unless explicit redistribution permission covers public open-source redistribution.

Local/private test use under a valid license is distinct from permission to publish fixtures in this repository.

## W3C tests and specifications

W3C test suites/specification materials may have their own licenses and attribution requirements. If W3C tests are vendored, preserve the applicable license and provenance rather than assuming project licensing.

Passing a W3C test suite must not be described as W3C certification or endorsement.

## External-oracle tests

CI jobs may download/reference external public corpora or run other processors without committing those corpora to this repository, provided:

- usage complies with the source terms;
- CI remains reproducible enough for its purpose;
- the external corpus is not silently republished as a release artifact;
- such jobs are clearly separated from the zero-dependency product runtime.

## Bug reports

Issue templates should ask users to provide minimal sanitized fixtures.

If reproduction requires confidential material, move the exchange to the private security/contact process before any file is shared.

## Review checklist

Before merging a third-party fixture:

- [ ] Canonical source identified.
- [ ] Exact version/revision identified.
- [ ] Redistribution rights reviewed.
- [ ] Required attribution included.
- [ ] Trademark/endorsement issues considered.
- [ ] Confidential/proprietary content excluded.
- [ ] Provenance metadata added.
- [ ] Fixture size is justified.
- [ ] A smaller synthetic fixture would not provide equivalent coverage.
