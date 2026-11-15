# Contributing to TelemetryWorks

Contributions of code, tests, documentation and bug reports are welcome.
This is the organization default for repositories without their own guide.
Repository-specific `CONTRIBUTING.md`, `AGENTS.md` and `CLAUDE.md` instructions
(including scoped files) take precedence. Read them, the README, manifests
and CI workflows before changing code.

## Propose a change

1. Check existing GitHub issues; open an issue in the affected repository to
   discuss substantial changes. For bugs, include the version, platform,
   expected/actual behavior and a minimal reproducer. Remove sensitive data.
   Report security issues privately using
   [SECURITY.md](https://github.com/TelemetryWorks/.github/blob/main/SECURITY.md),
   never in public issues or PRs.
2. Fork the repository (or use an authorized feature branch), branch from
   current `main`, and keep the change focused. Open a pull request to `main`
   explaining the change, linked issue, tests and results.
3. Obtain independent review, address feedback and pass CI and all required
   checks before merging. Main protection requires PRs; do not bypass it.
   Use merge commits, following the
   [security program](https://github.com/TelemetryWorks/.github/blob/main/SECURITY-PROGRAM.md)
   for protection details and current exceptions. Do not force-push main.

## Coding standards

- Rust defaults are edition 2024 and MSRV 1.85. Preserve the destination
  manifest's edition and MSRV; repository requirements override these defaults
  (for example, `jseries` requires 1.86). Do not raise them without approval.
- Format with rustfmt; run Clippy with `-D warnings` and fix warnings before
  merge. Keep the repository's compiler warnings and lint gates enabled.
- Add no new dependencies without maintainer approval. Do not add `unsafe`
  code where repository policy or lints deny it; preserve existing safety gates.

## Required tests and evidence

New functionality and bug fixes **MUST add automated tests**. Bug fixes must
include a regression test that fails before the fix and passes afterward.
Derive expected results from the applicable standard (cite edition/section)
or an independent oracle, not from the implementation being tested.
Follow tests-first development wherever the repository requires it.

Include test names, requirement/oracle references, commands and results in
the PR so reviewers can verify that tests were added and validate the change.
Reviewers must check this evidence before accepting the PR. Update test
instructions when adding a suite or changing how tests run. For documentation
changes, check links and examples as applicable.

Use the repository's documented test commands and full CI matrix, including
feature, platform and MSRV checks. For a typical Rust package, start with:

```sh
cargo fmt --all -- --check
cargo clippy --all-targets --locked -- -D warnings
cargo test --locked
```

These commands do not replace repository-specific checks. CI must pass; if
there is no CI yet, run the applicable checks locally and report the results.
Do not describe unexecuted checks as passing.

## Contribution licence

Contributions must use the destination repository's `LICENSE` and applicable
package licence. Rust crate repositories have Apache-2.0 root licences;
check each package as well. Current exceptions in manifests include MIT in
[`pelorus`](https://github.com/TelemetryWorks/pelorus/blob/main/Cargo.toml)
and MIT OR Apache-2.0 in
[`irig106-studio-core`](https://github.com/TelemetryWorks/irig106-studio/blob/main/crates/irig106-studio-core/Cargo.toml)
and [`irig106-studio-wasm`](https://github.com/TelemetryWorks/irig106-studio/blob/main/crates/irig106-studio-wasm/Cargo.toml).
Ask maintainers to resolve conflicting declarations before contributing;
this guide does not change any licence.
