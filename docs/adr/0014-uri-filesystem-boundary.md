# ADR-0014: URI-reference semantics remain separate from filesystem paths

- **Status:** Accepted
- **Date:** 2026-09-17

## Context

XSD `schemaLocation`, `xml:base`, and XML Catalog mappings are URI references. Host filesystem paths have different platform-specific semantics, especially on Windows. Mixing URI and filesystem operations can produce different schema graphs on different operating systems or incorrectly resolve references.

## Decision

URI/reference resolution and filesystem access are separate layers.

Rules:

1. XSD and catalog references are parsed and composed using URI-reference semantics.
2. `xml:base` is resolved in the URI layer.
3. Logical source identity uses normalized URI-style identifiers, not host absolute paths.
4. No `filepath.Join`, drive-letter interpretation, or platform path cleaning occurs while resolving XSD URI references.
5. Conversion from a resolved local logical reference to a filesystem request happens only at the `internal/source` boundary.
6. The source layer enforces the configured workspace root and rejects escapes.
7. Remote URI schemes are never dereferenced; they must map explicitly to a local source or resolution fails.
8. Host absolute paths embedded in untrusted schema input are not treated as authority to escape the configured source root.

## Consequences

- Linux, macOS, and Windows should resolve the same schema graph for the same logical source set.
- Source and URI code remain deliberately separate even when both ultimately refer to files.
- Cross-platform tests must cover relative references, `xml:base`, percent escaping, Windows-looking paths, catalog rewrites, and path traversal attempts.