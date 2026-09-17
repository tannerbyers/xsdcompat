# ADR-0001: Go and zero third-party product dependencies

**Status:** Accepted  
**Date:** 2026-09-17

## Context

The project is intended for local and CI use in environments where software supply-chain review, dependency approval, offline operation, and long-term maintenance matter. Existing XSD libraries either introduce broader runtime/dependency stacks, expose an API poorly suited to semantic diffing, are immature/archived, or require native dependencies.

Go provides a strong standard library, straightforward static binary distribution, cross-compilation, native fuzzing, and first-party vulnerability tooling.

The project does not require a complete XML Schema instance validator; it requires a bounded semantic compiler and conservative compatibility rule engine. That makes a stdlib-only implementation plausible without reimplementing every XSD subsystem.

## Decision

Implement `xsdcompat` in Go.

The shipped Go module and official binary will contain no third-party Go modules unless this ADR is superseded by a later dependency-exception ADR.

External development/CI tooling does not count as a product dependency provided it does not enter the distributed Go module/binary graph.

## Consequences

### Positive

- minimal SBOM and license-review surface;
- no transitive module vulnerabilities or abandoned dependencies;
- easier enterprise source/security review;
- predictable standalone binary distribution;
- project controls semantic model and API directly;
- fewer forced upgrades caused by upstream API changes.

### Negative

- project owns XSD semantic compiler correctness;
- namespace/QName handling, schema composition, derivation, groups, substitution groups, and catalog resolution require significant first-party tests;
- maintenance work that an upstream XSD library might otherwise perform becomes project responsibility.

## Boundary

The decision does **not** imply implementing a full XSD validator.

The project intentionally excludes validator-only subsystems such as general XML instance validation, arbitrary XPath execution, XSD regex execution, and universal content-language inclusion.

## Exception process

A future product dependency requires a superseding/exception ADR demonstrating that:

1. the capability is required;
2. implementing it internally creates greater security/correctness risk;
3. maintenance and release health are acceptable;
4. license and patent terms are acceptable;
5. the full transitive graph is understood;
6. relevant vulnerability/security history is reviewed;
7. dependency-specific types remain isolated from the public API;
8. a replacement/removal strategy exists.

Convenience alone is not sufficient.
