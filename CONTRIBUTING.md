# Contributing to xsdcompat

`xsdcompat` is correctness-sensitive infrastructure. Contributions are welcome, but compatibility behavior, parser changes, and dependency additions require evidence and focused tests.

## Before contributing

Please read:

- [DESIGN.md](DESIGN.md)
- [docs/COMPATIBILITY.md](docs/COMPATIBILITY.md)
- [docs/THREAT_MODEL.md](docs/THREAT_MODEL.md)
- relevant [architecture decisions](docs/adr/README.md)

For substantial behavioral or architectural changes, open an issue first so the design can be discussed before implementation effort is spent.

## Developer Certificate of Origin

Contributions use the [Developer Certificate of Origin 1.1](https://developercertificate.org/).

Sign each commit with:

```text
Signed-off-by: Your Name <you@example.com>
```

Git can add this automatically with:

```bash
git commit -s
```

By signing off, you certify that you have the right to submit the contribution under the project's Apache-2.0 license.

Do not contribute code, tests, schemas, documents, or other material copied from an employer, customer, standards body, or third party unless you have the right to do so and all required attribution/license information is included.

## Development principles

Contributions should preserve these project constraints:

- correctness over coverage;
- unsupported semantics fail toward `REVIEW`/`ERROR`, never optimistic compatibility;
- zero third-party Go modules unless an approved ADR supersedes that rule;
- no runtime network access;
- deterministic outputs;
- no package-level mutable state;
- bounded work for untrusted input;
- no user-input panic paths;
- semantic policy and enforcement policy remain separate.

## Pull requests

Keep pull requests small and focused. Do not mix parser behavior changes with unrelated refactors.

A behavioral PR should normally include:

- a clear problem statement;
- standards/reference reasoning when XSD semantics are involved;
- minimal old/new schema fixtures;
- tests for relevant directions (`backward`, `forward`, `full`);
- tests for the nearest unsupported or ambiguous boundary;
- documentation updates if a public behavior or rule changes.

Parser/compiler bug fixes should include a permanent regression fixture.

## Compatibility rules

A new rule that emits `BREAKING` or `NON_BREAKING` must document why the result is provable under the project's compatibility definition.

If that proof depends on assumptions not represented in the semantic model, the result should normally be `REVIEW` instead.

Stable rule IDs must not be repurposed for different semantics.

## Dependencies

Do not add a production Go module as part of an ordinary PR.

A proposed dependency requires an ADR satisfying the dependency-exception criteria in [DESIGN.md](DESIGN.md#33-dependency-exception-policy), including maintenance, licensing, security, transitive graph, API isolation, and removal strategy.

Development and CI tooling is evaluated separately and must not enter the shipped module graph.

## Third-party fixtures

Third-party schemas require explicit provenance and licensing records as described in [docs/FIXTURE_POLICY.md](docs/FIXTURE_POLICY.md).

Do not commit proprietary standards or customer schemas without documented redistribution rights.

Prefer minimal synthetic fixtures for individual parser/rule tests.

## Testing

Expected checks eventually include:

```bash
gofmt -w .
go vet ./...
go test ./...
go test -race ./...
govulncheck ./...
```

Fuzz and extended corpus jobs may run separately from normal PR checks.

Tests and golden outputs must not depend on map iteration order, wall-clock time, absolute checkout paths, or other machine-specific state.

## Commit and PR quality

Prefer commits and pull requests that are:

- small enough to review deeply;
- reversible;
- explicit about semantic changes;
- free of drive-by cleanup;
- backed by tests rather than comments alone.

## Security issues

Do not report exploitable vulnerabilities in a public issue. Follow [SECURITY.md](SECURITY.md).

## License

By contributing, you agree that your contribution is provided under the Apache License 2.0 and your DCO sign-off.
