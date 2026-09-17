# ADR-0012: XML input and encoding contract

- **Status:** Accepted
- **Date:** 2026-09-17

## Context

`xsdcompat` depends on correct XML tokenization and namespace/QName interpretation. Go's `encoding/xml` is a tokenizer, not a complete XML/XSD processor, and it does not transparently provide all encoding and namespace behavior required by XML Schema.

Ambiguous XML-version or character-encoding behavior would make schema compilation platform-dependent or silently incorrect.

## Decision

The v1 input contract is:

- XML 1.0 only;
- Namespaces in XML semantics required by XSD;
- UTF-8 and UTF-16 input support;
- UTF-16LE/UTF-16BE detection handled explicitly before `encoding/xml` tokenization;
- unsupported declared character encodings fail with a structured input error;
- DTDs and external entities in XSD input are rejected;
- XML Catalog documents may tolerate a catalog DOCTYPE declaration only as syntax; the runtime never retrieves or expands its external DTD;
- malformed names, duplicate attributes, invalid namespace bindings, invalid QNames, invalid XML characters, and inconsistent encoding declarations fail closed.

The scanner MUST NOT assume that successful `encoding/xml` token production alone establishes the complete XML/XSD lexical contract. Project-owned admission checks will cover semantics relied upon by the schema compiler.

## Consequences

- The project remains stdlib-only but must own a small UTF-16 decoding/admission layer.
- XML 1.1 is explicitly out of the v1 input contract.
- Encoding behavior becomes deterministic across operating systems.
- Conformance tests must cover BOM/declaration combinations, UTF-8/UTF-16, namespaces, QNames, invalid characters, duplicate attributes, and malformed declarations.