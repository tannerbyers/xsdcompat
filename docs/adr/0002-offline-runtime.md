# ADR-0002: Offline-only runtime

**Status:** Accepted  
**Date:** 2026-09-17

## Context

Target users may work with proprietary or regulated schema definitions and often run tools inside restricted CI/build environments. Remote schema resolution adds data-exfiltration, SSRF, reproducibility, availability, and operational-security risks that are unnecessary for the core problem.

## Decision

`xsdcompat` will be offline by architecture.

The runtime will not include automatic network schema retrieval, telemetry, update checks, or hosted-service dependencies.

Remote identifiers such as `http:`/`https:` schema locations may be mapped explicitly to local resources through supported catalog/source configuration, but the tool itself will not dereference them over the network.

## Consequences

### Positive

- no SSRF surface through XSD/catalog input;
- proprietary schemas remain local by default;
- deterministic builds and analysis are easier;
- CI does not depend on external availability;
- security review is simpler.

### Negative

- users must provide all required schema sources locally;
- workflows that rely on live remote schema locations need a preparation/mirroring step;
- XML Catalog/local mapping support becomes important.

## Scope

This decision applies to the product runtime. Documentation, CI, release automation, or development-oracle jobs may access external services when explicitly configured for those purposes.

Enabling runtime remote fetching requires a superseding ADR with an explicit threat model and trust-policy design.
