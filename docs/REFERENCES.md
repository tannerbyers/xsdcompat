# Authoritative References

This document records primary references that inform the project's architecture, security, standards scope, licensing, and release practices. It is not a bibliography of every discussion or implementation reviewed during research.

When project behavior conflicts with a normative standard, the relevant standard text and the project's explicitly documented support boundary take precedence over assumptions copied from other implementations.

## XML and XML Schema

- W3C XML Schema Definition Language (XSD) 1.1 Part 1: Structures  
  https://www.w3.org/TR/xmlschema11-1/
- W3C XML Schema Part 1: Structures Second Edition (XSD 1.0)  
  https://www.w3.org/TR/xmlschema-1/
- W3C XML Schema Part 2: Datatypes Second Edition  
  https://www.w3.org/TR/xmlschema-2/
- W3C XML 1.0  
  https://www.w3.org/TR/xml/
- W3C Namespaces in XML 1.0  
  https://www.w3.org/TR/xml-names/

## XML Catalogs

- OASIS XML Catalogs 1.1  
  https://www.oasis-open.org/standard/xmlcatalogs/

Catalog support in `xsdcompat` may intentionally implement only a documented subset. The project must not claim full XML Catalog conformance merely because selected mappings are supported.

## Representative schema ecosystems

- ISO 20022 message catalogue  
  https://www.iso20022.org/iso-20022-message-definitions
- ISO 20022 terms of use / IPR material  
  https://www.iso20022.org/terms-use
- NIEM Naming and Design Rules  
  https://niemopen.github.io/niem-naming-design-rules/
- NIEM model repository  
  https://github.com/niemopen/niem-model
- NIEM migration tooling documentation  
  https://niem.github.io/reference/tools/migration/
- FDA Structured Product Labeling resources  
  https://www.fda.gov/industry/fda-data-standards-advisory-board/structured-product-labeling-resources

These ecosystems are used as representative design/test inputs where licensing permits; `xsdcompat` is not affiliated with or certified by them.

## Go security and runtime

- Go security best practices  
  https://go.dev/doc/security/best-practices
- Go vulnerability database and `govulncheck`  
  https://go.dev/doc/security/vuln/
- Go release/security support information  
  https://go.dev/doc/devel/release
- Go `os` package (`Root` / rooted filesystem APIs)  
  https://pkg.go.dev/os
- Go `encoding/xml` package  
  https://pkg.go.dev/encoding/xml
- Go fuzzing  
  https://go.dev/doc/security/fuzz/

## AWS engineering/security principles

The project is not an AWS workload, but its engineering controls intentionally borrow relevant public principles from AWS Well-Architected:

- AWS Well-Architected Security Pillar  
  https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/security.html
- AWS Well-Architected Reliability design principles  
  https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/design-principles.html
- AWS Well-Architected Operational Excellence design principles  
  https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/design-principles.html
- AWS software supply-chain security guidance  
  https://aws.amazon.com/blogs/security/well-architected-best-practices-for-software-supply-chain-security/

Relevant principles include least privilege, traceability, automated security controls, small reversible changes, treating operations as code, anticipating failure, and validating software/build integrity.

## Secure software development

- NIST Secure Software Development Framework (SSDF), SP 800-218  
  https://csrc.nist.gov/publications/detail/sp/800-218/final
- OpenSSF concise guidance for developing more secure software  
  https://best.openssf.org/Concise-Guide-for-Developing-More-Secure-Software.html

The project does not claim formal NIST/OpenSSF certification; these are engineering references.

## Licensing and contributions

- Apache License 2.0  
  https://www.apache.org/licenses/LICENSE-2.0
- Developer Certificate of Origin 1.1  
  https://developercertificate.org/
- SPDX specification  
  https://spdx.dev/use/specifications/

## GitHub release provenance

- GitHub Artifact Attestations  
  https://docs.github.com/en/actions/concepts/security/artifact-attestations
- GitHub Actions security hardening  
  https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions

## Dependency and implementation research

Existing processors may be used as development oracles but are not normative authorities. Projects reviewed during initial design include:

- `jacoelho/xsd` — pure-Go XSD implementation  
  https://github.com/jacoelho/xsd
- `lestrrat-go/helium` — broader Go XML/XSD stack  
  https://github.com/lestrrat-go/helium
- `droyo/go-xml` — historical Go XSD/code-generation implementation and issue corpus  
  https://github.com/droyo/go-xml
- Python `xmlschema`  
  https://github.com/sissaschool/xmlschema
- Apache Xerces-J  
  https://xerces.apache.org/xerces2-j/
- Apicurio Registry XSD compatibility implementation  
  https://github.com/Apicurio/apicurio-registry

Disagreement between external implementations is evidence to investigate against standards and focused fixtures, not a reason to copy one implementation's behavior blindly.
