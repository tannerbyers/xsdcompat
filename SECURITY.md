# Security Policy

## Supported versions

`xsdcompat` is currently pre-release. Until 1.0, only the latest development release is expected to receive security fixes.

After 1.0, supported release lines will be documented here. The project does not currently promise an enterprise support SLA or long-term support branch.

## Security contact

The current security contact is the repository maintainer, [@tannerbyers](https://github.com/tannerbyers).

Please do **not** send confidential vulnerability details through a public issue, discussion, or pull request.

## Reporting a vulnerability

Please do **not** open a public issue for a suspected vulnerability that could put users at risk.

Preferred reporting method: use GitHub's private vulnerability reporting / Security Advisory flow for this repository when available.

If that mechanism is unavailable, open a minimal public issue asking `@tannerbyers` for a private contact channel without including exploit details, sensitive schemas, credentials, or proof-of-concept payloads.

Useful reports include:

- affected version or commit;
- operating system / architecture when relevant;
- minimal reproduction or sanitized fixture;
- impact and attack preconditions;
- whether untrusted XSD/catalog input is required;
- any known workaround.

## Response targets

The following are best-effort open-source project targets, not contractual SLAs:

- acknowledge a private security report within **7 calendar days** where practicable;
- provide an initial triage/status response within **14 calendar days** where practicable;
- keep the reporter informed when a confirmed issue requires longer remediation;
- coordinate public disclosure after a fix or mitigation is available when reasonable.

If maintainer availability prevents those targets from being met, the project makes no warranty that a specific response or fix time will be achieved.

## Security scope

The project treats the following classes as security-sensitive:

- path traversal or symlink escape outside the configured workspace;
- external entity or unexpected DTD processing;
- network access triggered by schema/catalog input;
- parser/compiler panic from untrusted input;
- uncontrolled recursion or algorithmic denial of service;
- excessive memory allocation from crafted schemas;
- namespace/QName confusion that changes semantic interpretation;
- catalog mappings escaping the local trust boundary;
- output leaking unexpected absolute host paths or file contents;
- release/supply-chain tampering;
- vulnerabilities in the Go runtime/standard library affecting the tool.

Purely incorrect compatibility classifications are normally correctness bugs, but a false `NON_BREAKING` result on unsupported or malformed semantics is treated with security-like severity because enterprise automation may trust that classification.

## Security design

The planned runtime intentionally minimizes attack surface:

- zero third-party Go modules in the shipped product;
- no network resolver;
- no telemetry or update check;
- no plugin runtime;
- no command/shell execution hooks;
- workspace-bounded filesystem access;
- no external entity processing;
- explicit work/resource limits;
- deterministic structured diagnostics;
- conservative `REVIEW` behavior for uncertain semantics.

The configured workspace is a file-access trust boundary, not an operating-system sandbox. Callers remain responsible for not exposing sensitive mounts, special files, devices, or other host resources inside that workspace.

See [docs/THREAT_MODEL.md](docs/THREAT_MODEL.md) for detailed threats and controls.

## Coordinated disclosure

The project will make a best-effort attempt to:

1. acknowledge a valid private report;
2. reproduce and assess impact;
3. prepare a fix and regression test;
4. coordinate an appropriate disclosure date with the reporter;
5. publish a patched release and advisory when warranted.

The response targets above are project goals, not a paid support or enterprise SLA.

## Go toolchain security

Zero third-party dependencies does not mean zero upstream vulnerabilities. Official builds should use a currently security-supported Go toolchain, and CI/release workflows should run `govulncheck` or the current official equivalent.

The project follows Go's published security support model and should rebuild affected binaries when a relevant Go standard-library/runtime vulnerability is fixed.

## Sensitive test data

Do not attach proprietary, regulated, or confidential schemas to public issues unless you have permission to disclose them.

Prefer a minimized synthetic reproducer. If a bug only reproduces on sensitive material, coordinate privately before sharing it.
