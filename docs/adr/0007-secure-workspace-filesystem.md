# ADR-0007: Workspace-Bounded Filesystem Access

**Status:** Accepted

## Context

XSD and XML Catalog inputs can contain relative paths and URI references. A local-only tool can still be vulnerable to path traversal or symlink escape if untrusted schemas are allowed to resolve arbitrary host paths.

## Decision

The CLI operates inside an explicit workspace root and must use a traversal-resistant filesystem boundary (`os.Root`/`Root.FS` when available for the supported Go version, or an equivalently proven design).

Schema/catalog resolution must not escape that root. Unmapped network locations are never fetched. The library should prefer caller-supplied `fs.FS`/source abstractions rather than implicitly opening arbitrary process paths.

## Consequences

- Schema inputs cannot silently read unrelated host files through `..`, absolute paths, or symlink tricks.
- Tests must include traversal and symlink adversarial cases.
- Some historical schema layouts that depend on arbitrary host-relative paths may need users to stage files under a workspace or map them through a supported catalog.

## Rejected alternatives

- Plain unrestricted `os.Open`: unnecessarily broad trust boundary.
- `os.DirFS` alone as a security boundary: pathname confinement properties are not sufficient for hostile-path use cases.
- Remote fetching as fallback: conflicts with offline and SSRF guarantees.
