# Security Policy

This policy covers every TelemetryWorks repository that does not carry its
own `SECURITY.md`.

See the [organization security program](SECURITY-PROGRAM.md) for the controls
inventory, publishing policy and planned release assurances.

## Reporting a vulnerability

Please report suspected vulnerabilities **privately**, through GitHub's
private vulnerability reporting:

1. Open the affected repository on GitHub.
2. Go to **Security** → **Advisories** → **Report a vulnerability**.
3. Describe the problem there. Only the maintainers can see the report.

Please **do not** open a public issue, pull request, or discussion about a
security problem.

### What to include

When you can:

- a description of the vulnerability and its potential impact;
- the affected repository, crate, and versions;
- the input or steps that reproduce it (for these projects often a small
  file or byte sequence);
- a proof of concept, if you have one.

### What to expect

- We aim to acknowledge a report within **7 days**.
- We ask for at least **30 days** to fix a confirmed problem before it is
  disclosed publicly, and we will agree the details of publication with you
  once a fix is available.
- We credit reporters who want to be credited.
- We will not pursue legal action against good-faith security research.

## Supported versions

Security fixes go into:

- the most recent release of each crate or tool, and
- the default branch.

Older releases are fixed on a best-effort basis, depending on severity.

## How we handle a confirmed problem

1. Assess its severity and scope.
2. Develop and test a fix or mitigation, keeping the reporter informed.
3. Release fixed versions.
4. Publish a GitHub security advisory, with a CVE where appropriate.

## Security practices

Each repository states its own practices in its documentation. For the
IRIG 106 crates (`irig106-*`) they are:

| Practice | Status |
|----------|--------|
| No `unsafe` code | required |
| Formatting, Clippy with warnings as errors, and tests on every change | in place in `irig106-tmats` and `irig106-time` |
| Dependency policy checked by `cargo deny` (known vulnerabilities, unmaintained or yanked crates, licences, sources) | in place in `irig106-tmats` and `irig106-time` |
| Clippy lints against `unwrap`, `expect`, panics, and printing in library code | planned, with the rebuilt libraries |
| Tests on Windows, Linux, and macOS | in place in `irig106-tmats` and `irig106-time` |
| Minimum test coverage | planned |
| Fuzzing of parsers | planned for the rebuilt libraries (`irig106-time` has fuzz harnesses from its prototype, not yet run in CI) |
| Software bill of materials and build-provenance attestation for released binaries | planned, with the first releases of binaries |
| Publishing to crates.io from GitHub Actions via Trusted Publishing | required by owner policy 2026-11-13; owner decision 2026-11-15 allows One-time token bootstrap for a brand-new crate’s first upload only: crate-scoped `publish-new` token, 1-day expiry, revoke, then Trusted Publisher + Trusted-Publishing-only mode; per-crate evidence pending — see [security program](SECURITY-PROGRAM.md#publishing-policy) |

Released binaries will be verifiable with
`gh attestation verify <file> --owner TelemetryWorks` once attestations are
in place.
