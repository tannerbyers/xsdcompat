# Compatibility Contract

This document defines what `xsdcompat` means by compatibility and how classifications must be produced.

## Scope

`xsdcompat` evaluates **XSD-level instance-language compatibility** for supported XSD semantics.

It does not prove compatibility of:

- application code;
- generated JAXB/.NET/etc. bindings;
- database models;
- XSLT or other transformations;
- Schematron/business rules;
- usage guidelines;
- external code lists;
- regulatory submission rules;
- service behavior.

A successful XSD result must not be presented as an end-to-end migration guarantee.

## Directions

Let `L(S)` represent the set of XML instances valid under schema set `S`, restricted to semantics the engine supports.

### Backward compatibility

A new schema is backward compatible with the old schema when:

```text
L(old) ⊆ L(new)
```

Existing valid instances remain valid under the new schema.

### Forward compatibility

A new schema is forward compatible with the old schema when:

```text
L(new) ⊆ L(old)
```

Instances produced for the new schema remain valid under the old schema.

### Full compatibility

Full compatibility requires both backward and forward compatibility.

## Classifications

### `BREAKING`

A supported rule proves that the change violates the requested compatibility direction.

Example for backward compatibility:

```text
minOccurs: 0 -> 1
```

An old instance may omit the particle, while the new schema requires it.

### `NON_BREAKING`

A supported rule proves compatibility for that change and direction.

This classification may only be emitted when the rule has sufficient semantic information to make the proof.

### `REVIEW`

The engine detected a meaningful semantic change but cannot safely prove either outcome.

Examples expected to default to review until specifically supported:

- arbitrary pattern replacement;
- complex content-model restructuring not covered by a proven rule;
- XSD 1.1 assertions;
- XSD 1.1 conditional type alternatives;
- difficult union/value-space changes;
- identity-constraint expression changes where execution/inclusion is not modeled.

`REVIEW` is not an error and is not equivalent to non-breaking.

### `ERROR`

The schema set could not be compiled into a trustworthy semantic graph or analysis could not complete safely.

Examples:

- unresolved required source;
- malformed XML/XSD preventing semantic construction;
- unsupported `xs:redefine` in a position that changes component construction;
- resource/work budget exceeded;
- internal invariant failure.

An `ERROR` must not be converted into a compatibility verdict.

## Overall result precedence

After successful compilation:

```text
if any finding is BREAKING:
    overall = BREAKING
else if any finding is REVIEW:
    overall = REVIEW
else:
    overall = NON_BREAKING
```

Operational/compiler failure yields `ERROR` instead.

## Policy versus semantic truth

Semantic classification and enforcement disposition are separate.

A policy may waive a breaking change:

```text
classification: BREAKING
disposition: WAIVED
```

It must never rewrite the classification to `NON_BREAKING`.

This distinction is required for auditability and reproducibility.

## Rule requirements

Every rule that emits `BREAKING` or `NON_BREAKING` must have:

1. a stable rule ID;
2. documented reasoning;
3. fixtures for the relevant compatibility directions;
4. boundary tests showing where the rule stops applying;
5. deterministic output;
6. no assumptions about application behavior that are absent from the semantic model.

If a proposed rule depends on an unmodeled assumption, it should normally emit `REVIEW` instead.

## Content-model limitation

In the general case, XSD model groups define languages of allowed element sequences. `xsdcompat` does not attempt a universal content-language inclusion theorem prover.

The rule engine proves selected transformations where the inclusion relation is known. More complicated changes remain `REVIEW` until a bounded, tested decision procedure is intentionally added.

## Value-space limitation

XSD lexical forms and value spaces are not always equivalent to string comparison. Rules involving numeric ranges, defaults/fixed values, enumerations, dates/times, QName values, floating special values, or other typed data must only classify automatically when the engine implements the required value semantics.

Otherwise the result is conservative.

## Rule and report versioning

Reports will identify at least:

- `toolVersion`;
- `rulesetVersion`;
- `formatVersion`;
- `configVersion`.

A classification correction can materially change CI behavior even when the CLI/API shape does not change. Such changes must be called out in release notes and reflected in the ruleset-version policy established before 1.0.
