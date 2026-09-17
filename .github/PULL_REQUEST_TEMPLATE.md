## Summary

Describe the change and why it is needed.

## Type of change

- [ ] Documentation / repository maintenance
- [ ] Parser / schema compiler
- [ ] Semantic normalization / diff
- [ ] Compatibility rule
- [ ] Security / resource-limit behavior
- [ ] Public API / CLI / report format
- [ ] Release / build / CI
- [ ] Other

## Correctness evidence

For semantic or compatibility changes, explain the relevant XSD semantics and provide a standards/reference rationale where applicable.

## Tests

- [ ] Existing tests pass.
- [ ] New behavior has focused tests.
- [ ] Parser/compiler bug fixes include a regression fixture.
- [ ] Compatibility-rule changes cover relevant backward/forward/full directions.
- [ ] Nearby unsupported/ambiguous behavior still returns `REVIEW` or `ERROR` rather than an optimistic result.

## Security and maintenance review

- [ ] No new runtime network access.
- [ ] No unbounded input-derived recursion/allocation/work was introduced.
- [ ] No unexpected host paths or schema contents are logged/output.
- [ ] No third-party Go module was added, or an approved ADR is included.
- [ ] Third-party fixtures include required provenance/license information.

## Public contract

- [ ] No public contract changes, or documentation/versioning implications are described below.

Potentially affected contracts:

- Go API
- CLI arguments/commands
- configuration format
- JSON/SARIF report schema
- rule IDs/classifications
- exit behavior

## DCO

- [ ] Commits include `Signed-off-by` DCO sign-off.

## Notes for reviewer

Call out anything that deserves especially careful review.
