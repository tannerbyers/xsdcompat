# ADR-0013: Strict configuration parsing

- **Status:** Accepted
- **Date:** 2026-09-17

## Context

Configuration can affect CI enforcement, compatibility direction, source resolution, resource limits, and waivers. Silently accepting misspelled or unknown configuration would be especially dangerous because a user could believe a control is active when it is not.

The project uses JSON initially to preserve the zero-third-party-module constraint.

## Decision

Configuration is fail-closed.

The parser MUST reject:

- unsupported `configVersion` values;
- unknown object fields;
- duplicate JSON object keys;
- invalid enum values;
- type mismatches;
- malformed waiver entries;
- values outside documented bounds;
- ambiguous or conflicting options.

Unknown fields MUST NOT be ignored for forward compatibility. A future configuration format change requires an explicit version change when old parsers cannot interpret it safely.

Configuration MUST NOT support:

- environment-variable interpolation;
- embedded scripts;
- executable hooks;
- runtime plugins;
- implicit network references.

## Consequences

- Configuration mistakes fail early instead of weakening enforcement silently.
- Parsing may require project-owned duplicate-key detection in addition to normal `encoding/json` decoding.
- New configuration fields must be introduced deliberately with compatibility/versioning consideration.
- Tests must include unknown fields, duplicate keys, unsupported versions, invalid enums, overflow/boundary values, and conflicting settings.