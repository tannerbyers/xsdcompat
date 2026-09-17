# ADR-0005: CLI and Versioned JSON as the Portable Public Contract

**Status:** Accepted

## Context

`xsdcompat` is intended for heterogeneous enterprise environments. Consumers may use Java, .NET, Python, Go, shell, Jenkins, GitLab, Azure DevOps, or internal platform tooling. A Go-only API would unnecessarily couple adoption to the implementation language.

## Decision

The portable public integration contract is the standalone CLI plus a versioned JSON report format. A small Go library API may also be provided for Go callers.

The JSON report must identify at least the tool, report-format, configuration, and ruleset versions once those contracts exist.

Human-readable text, Markdown, and SARIF are projections of the same semantic result and must not introduce separate classification logic.

## Consequences

- Consumers do not need Go installed when using released binaries.
- CI and other languages can integrate through process exit behavior and JSON.
- JSON/report compatibility becomes a release-management obligation at 1.0.
- Internal Go types must not accidentally become the portable format.

## Rejected alternatives

- Go library as the only public API: too restrictive for the target environments.
- Hosted API/service as the primary interface: conflicts with offline/privacy goals.
- Language-specific SDKs in v1: unnecessary maintenance surface.
