# Standard release pipeline: `trusted-publishing.yml`

The owner approved this standard on 2026-11-22.

This adopts the structure of the [Rust Commons release-pipeline standard](https://github.com/rustcommons/.github/blob/6eb3d80401382ee197e58844f6008d53e406efe6/docs/TRUSTED-PUBLISHING.md)
for TelemetryWorks. It implements the direction of
[SECURITY-PROGRAM.md#publishing-policy](../SECURITY-PROGRAM.md#publishing-policy);
it records a standard and rollout state, not completed adoption or permission
to release. The security program's dated snapshots remain historical evidence.

Publishing uses `.github/workflows/trusted-publishing.yml`, with workflow
`name: Trusted publishing`. Register the repository, **filename**
`trusted-publishing.yml` and environment `crates-io` with crates.io. Renaming
jobs does not alter that identity; changing the registered filename does.
PyPI uses the same workflow filename and environment `pypi`.

## The pipeline

The crate-only pipeline has these six jobs, with exactly these ids and names.
Each job explicitly grants `contents: read` unless the table replaces it with
`contents: write`; no other permissions are granted beyond those listed.

| # | Job id | Display name | Runs on | Permissions |
| --- | --- | --- | --- | --- |
| 1 | `gate` | Release gate | Every run | `contents: read`, `actions: read` |
| 2 | `build` | Build | Every run | `contents: read` |
| 3 | `attest` | Checksums and attestations | Publishing tag runs | `contents: read`, `id-token: write`, `attestations: write` |
| 4 | `publish` | Publish to crates.io | Publishing tag runs, after owner approval | `contents: read`, `id-token: write`; environment `crates-io` |
| 5 | `github-release` | GitHub release | Publishing tag runs | `contents: write` |
| 6 | `verify` | Verify publication | Publishing tag runs | `contents: read` |

Dependencies are fixed: `build` needs `gate`; `attest` needs `[gate, build]`;
`publish` needs `[gate, build, attest]`; `github-release` needs `[gate, publish]`;
`verify` needs `[gate, github-release]`. Jobs 3–6 require
`needs.gate.outputs.publish == 'true'`. Keep `gate` directly in each dependent
job's `needs`, since outputs are visible only through direct dependencies.
The Python and binary variants below explicitly describe their extensions.

### 1. Release gate

Before a tag run builds anything, require all of the following:

- An **annotated** `v*` tag whose name is exactly `v` plus the release version.
- The tagged commit is reachable from `main` and has successful CI for that
  exact commit. Require the intended CI workflow, not an unrelated green run.
- Include every workspace crate at the release version in the ordered release
  set. Any other publishable workspace crate must already exist on crates.io
  at its declared version. Internal `publish=false` helpers are not uploads.
- A dated `## [X.Y.Z] - YYYY-MM-DD` entry in `CHANGELOG.md`, plus committed
  `docs/releases/vX.Y.Z.md`. Release notes derive from that changelog entry;
  missing or undated records fail the gate.

For `pull_request` and `workflow_dispatch`, report the findings and emit
`publish=false` so the build dry run can proceed without publishing. Outputs
are `version`, `publish` and the crate list in dependency order. Apply the
lockstep-workspace requirements below when defining that list.

### 2. Build

Run `cargo package --locked` for each release crate in dependency order,
including the build from its packaged source. Generate a CycloneDX SBOM with
an exact, pinned `cargo-cyclonedx` version installed using `--locked`.
Create `SHA256SUMS` covering each `.crate` and the SBOM. Upload one logical
release artifact named `release-<version>` containing those files.

All additional qualification, package audits and candidate-install tests
belong here and must also execute on the PR dry run. The Python variant builds
its wheels and sdist here too; publishing jobs never build them. The binary
variant uses a reusable builder as described below.

### 3. Checksums and attestations

Validate `SHA256SUMS` before signing. Produce GitHub artifact attestations
using Sigstore and SLSA build provenance for every `.crate` and the SBOM,
and for additional release files in the applicable variant. Retain the signed
bundle as `<repo>-<version>.crates.intoto.jsonl` with the release artifact.
Later jobs consume this frozen evidence; they must not substitute other bytes.
A provenance statement alone does not establish a SLSA level.

### 4. Publish to crates.io

Only this job uses the `crates-io` environment, where the owner approves
publication. Revalidate the downloaded artifact with `SHA256SUMS`.

**Nothing compiles in this job.** Its OIDC permission is available to code
throughout the job, not just after an authentication step. Reproduce each
archive with `cargo package --locked --no-verify`, then `cmp` it against the
attested `.crate`. Compilation and qualification already happened in `build`.
Do not run a compiling publish dry run in this job.

After comparison, obtain the short-lived credential with the full-SHA-pinned
`rust-lang/crates-io-auth-action`. Run `cargo publish --locked --no-verify`
in dependency order, exposing `CARGO_REGISTRY_TOKEN` from that action on the
publish step alone. This is an ephemeral OIDC credential, never a stored token.
Skip versions already on crates.io; confirm their checksum matches the expected
artifact and fail on a mismatch. This permits controlled recovery after a
partial upload without attempting to replace any published version.

### 5. GitHub release

Create `vX.Y.Z` using `--verify-tag` and the committed
`docs/releases/vX.Y.Z.md` as the release body, with its first heading as the
title. Attach the `.crate` files, SBOM, `SHA256SUMS` and provenance bundle
(and variant-specific files). A retry may upload missing assets only;
**never replace an asset**. Compare any existing asset to the expected bytes
and stop on a mismatch. The binary variant adds draft/resume verification.

### 6. Verify publication

- Check that each crates.io checksum equals the SHA-256 of its attested `.crate`.
- Run `gh attestation verify` for each attested attached file, constraining the
  expected repository and `trusted-publishing.yml` signer workflow. Validate
  the checksum manifest and bundle as evidence, not as claims that they
  themselves necessarily have individual attestations.
- Require successful docs.rs builds. Use a bounded wait; report a queue still
  pending at the end as **pending**, never passed.
- For published binary crates, use a clean `cargo install --locked` of the
  exact version into an empty root and check the executable's version.
  For binary-only distributions without a registry crate, verify and smoke-test
  the signed release archive instead; record that crates.io/docs.rs checks
  are not applicable, rather than claiming they passed.

## Common settings

- `push` tags `['v*']` invokes publishing. `pull_request`, filtered to workflow
  and release-script changes, and `workflow_dispatch` invoke only the gate and
  build dry run. Neither dry-run route signs, deploys or publishes.
- Top-level permission is `contents: read`. Grant additional permissions only
  to the individual job in the table; explicit job permissions include the
  required read access, since unspecified permissions become unavailable.
- Use `concurrency.group: trusted-publishing-${{ github.ref }}`. Set
  `cancel-in-progress` true only for PRs; never cancel a tag run for concurrency.
- Use `ubuntu-24.04` runners for the base jobs. Platform matrices belong inside
  qualified Python/binary builds when needed. Pin actions to full commit SHAs
  with version comments; checkout has `persist-credentials: false`.
- Install tools at exact versions with `--locked` where supported; Python tools
  use pinned, hash-locked requirements. Do not use `[skip ci]`.

### First-upload bootstrap exception

The existing [owner decision 2026-11-15](../SECURITY-PROGRAM.md#publishing-policy)
states:

> Owner decision 2026-11-15: **One-time token bootstrap** is the sole exception
> to the 2026-11-13 policy, for a brand-new crate's first upload only. Use a
> crate-scoped `publish-new` token with a 1-day expiry. After that upload, revoke
> the bootstrap token, configure the Trusted Publisher, and enable
> Trusted-Publishing-only mode before any subsequent upload. This exception
> does not permit token-based updates to an existing crate.

The coordinator/owner may perform only the brand-new name's **first upload**,
a name-placeholder `0.0.1` publish, with that one-time, crate-scoped,
`publish-new` token expiring in one day. Revoke it immediately afterward,
register the Trusted Publisher, and enable crates.io **Trusted Publishing only**
before any later upload. CI never performs the bootstrap and never stores a
token. An already-published placeholder cannot use this exception again.
This document authorizes no bootstrap or credential handling by this PR.

### Lockstep workspaces

A library-plus-CLI family releases the whole publishable workspace as one
ordered set, library before CLI, at `X.Y.Z`. The CLI must pin its library at
`=X.Y.Z` (with the local path for workspace development). This includes the
`irig106-time` and `irig106-tmats` families, with library paths under `crates/`
and CLI applications under `apps/` in the target layout. Keep internal fuzz
and helper packages out of the published set; a release-preparation PR must
resolve any CLI `publish=false` that would prevent the authorized family release.

Every family crate must be packaged, checksummed, attested, uploaded and verified;
no library-only shortcut. Bump all applicable lockfiles and exact pins together.
Current evidence: time already uses `crates/irig106-time` and
`apps/irig106-time-cli`, whose manifest still has `publish=false`; tmats still
uses the root library and `irig106-tmats-cli/`, with a CLI pin of `=0.1.0`.
Do not describe the tmats layout migration as complete. Adoption belongs in
release preparation: **irig106-time S20**, **irig106-tmats family 12**.

### Registry and environment checklist — complete for each adopting repository

The owner verifies these settings before the next authorized tag; this docs PR
neither changes them nor certifies their present configuration.

- For **each crates.io crate**, register the GitHub Trusted Publisher with owner
  `TelemetryWorks`, the exact destination repository, workflow filename
  `trusted-publishing.yml`, and environment `crates-io`. Turn **Trusted Publishing
  only ON** and verify `crate.trustpub_only == true` in the crates.io API.
  Existing published crates never fall back to a token; a stale filename or
  environment must be corrected before tagging.
- For **each PyPI project**, register the GitHub Trusted Publisher with owner
  `TelemetryWorks`, the exact repository, workflow `trusted-publishing.yml`, and
  environment `pypi`. Use OIDC publishing only. PyPI does not expose crates.io's
  `crate.trustpub_only` field; do not claim that toggle exists on PyPI.
- Protect the applicable GitHub environments `crates-io` / `pypi`: the owner is
  a required approver, bypass of environment protection is disabled, and
  deployment eligibility is restricted to `v*` **tags**, not branches. Verify
  protection explicitly; naming an environment in YAML alone does not protect it.
- Confirm the workflow registration, owner approvals and release-set membership
  for every library and CLI; do not assume one crate's configuration covers its
  sibling or the Python package. Record the completed checklist in the adoption PR.

## Repositories that also publish Python wheels

**Owner decision: ONE `v*` tag releases Rust and Python together. There are no
dual tags.** Crate, Python project, native binding and committed notes must
agree on the release version. Neither registry upload authorizes a second tag.

Keep the six base job ids and display names. Add job `publish-pypi`, display
name **Publish to PyPI**, with `needs: [gate, build, attest, publish]`, the same
publishing-tag condition, environment `pypi`, and only `contents: read` plus
`id-token: write`. The only dependency extension to the base release sequence
is `github-release` needing `[gate, publish, publish-pypi]` so assets are not
released before both registries finish. `verify` keeps `[gate, github-release]`
and verifies both registries' actual files.

Build and qualify wheels and sdist in `build`, add them to the same
`release-<version>` artifact and `SHA256SUMS`, and sign their GitHub provenance
before upload. In `publish-pypi`, recheck checksums and stage only those
prebuilt files. Use full-SHA-pinned `pypa/gh-action-pypi-publish` with
`attestations: true` for **PEP 740 per-file attestations**. Nothing compiles
there. Keep the registry identity at `trusted-publishing.yml` / `pypi`.
Validate PyPI file hashes and per-file attestations, then test clean installed
packages in the intended runtime/platform matrix. Preserve source-graph SBOM
scope limits; it is not a claim of complete native binary dependency coverage.
Native wheels also require the reusable-builder signing boundary below.

### Spatial-typestate: observed shape, not adopted yet

The inspected [workflow](https://github.com/TelemetryWorks/spatial-typestate/blob/4011671426926f3ea31fe5573a73876c613e0c5e/.github/workflows/trusted-publishing.yml)
is named **Coordinated release**, triggered by `v*.*.*`, and currently runs
`source`, `verify-crate`, `verify-python`, `verify-windows-wheel`, `verify`,
`publish-crates`, `publish-pypi`, `release-assets`.

It qualifies Rust and a Linux/Windows Python 3.11/3.14 matrix, selects two
`cp311-abi3` wheels (manylinux_2_28_x86_64 and win_amd64) plus the canonical
Linux sdist, and stages those three files alongside the `.crate` and SBOMs.
Its crate and PyPI jobs use separate `crates-io` / `pypi` approvals; PyPI
already uses pinned `pypa/gh-action-pypi-publish` with per-file PEP 740
attestations. Publishing is restricted to the original run (`run_attempt == 1`),
and `release-assets` is a third, non-OIDC `crates-io` approval.

This is not the new six-job pipeline: the current crate publish job compiles,
attestations happen in publishing jobs, the artifact is `release-package`,
and there is no PR dry-run trigger. Migration must preserve its qualification
and file-selection controls while moving compilation out of privileged jobs
and adopting the job structure and reusable signing boundary above.
The historical `publish-0.1.0.yml` is **RETIRED**: never enable, invoke or copy
its token-based route. Its continued presence is not an approved alternative.

## Repositories that also ship binaries

Every repository shipping binaries uses the binary variant, including a
binary-only GitHub release such as `irig106-ch10-reader`. This follows the
Rust Commons reusable-builder boundary: the build must attest its **own output**,
not a caller-selected artifact. It is a design requirement toward stronger
SLSA assurance, not a declaration that TelemetryWorks has achieved Build L3.

- The caller's `build` invokes `./.github/workflows/build-binaries.yml`, which
  builds and qualifies each target through a read-only `build-binaries-core.yml`,
  packages release crates and binaries, generates SBOMs and one `SHA256SUMS`,
  and signs its own verified output in a nested **Checksums and attestations**
  job only on a `v*` tag push. The caller cannot supply arbitrary signing inputs.
- There is **no top-level `attest` job** in this variant. `build` retains
  `needs: gate` and grants `contents: read`, `id-token: write` and
  `attestations: write` to the reusable workflow; only its nested signer uses
  the write permissions. Pass the gate's publishing decision explicitly.
  PR/manual runs build everything but never sign. `publish` needs `[gate, build]`
  instead of `[gate, build, attest]`; other base edges stay the same.
- All building and signing finish before the owner's registry approval. For
  native wheels combine this boundary with the Python variant: `publish-pypi`
  needs `[gate, build, publish]`, with signing completed inside `build`.
- `github-release` first creates or resumes a draft. Verify its tag, title,
  exact committed body and draft state, upload all assets, check each name,
  size and digest, publish, and check again. Never modify a published release.
  Resume a draft through the releases list rather than relying on tag lookup.
- The release contains `.crate` files where applicable, all binary archives,
  SBOMs, one `SHA256SUMS`, and one `<repo>-<version>.intoto.jsonl` bundle for
  all signed files. For binary-only repositories, registry publication is not
  applicable: retain the pipeline's job ids, make `publish` an explicit
  non-publishing completion gate, and verify the archives instead.
- Keep verifiers compatible with previously published layouts. Enforce the new
  layout only for later releases; do not rebuild, replace or re-tag old releases.

## Lessons from the rollout

- **CI lookup:** the workflow-runs API's `status=success` filter can miss green
  CI. Read all pages of runs for the tagged SHA (`?head_sha=SHA&per_page=100`)
  and identify the intended CI workflow with `conclusion == 'success'`.
- **Fuzz lockfiles:** update `fuzz/Cargo.lock` and any exact `=X.Y.Z` dependency
  in `fuzz/Cargo.toml` with the release bump, or locked fuzz checks can fail.
- **Binary signing:** a generic signer of caller-selected files loses the
  qualified-build linkage. Keep signing inside the reusable builder. A killed
  `gh release create` may leave a draft invisible to tag lookup; list and resume
  drafts deliberately, verifying their content before publication.
- **Smoke tests:** under `pipefail`, piping program output into `head` can cause
  a broken-pipe failure. Capture output in a file before inspecting it.
- **Check registries before recovery:** types run `43727921675` published 0.1.0
  in attempt 2; attempt 3 failed because it already existed (see the
  [historical evidence](../SECURITY-PROGRAM.md#verified-release-and-attempt-history)).
  Never rerun after a successful upload merely to make a display green. A new
  pipeline's authorized partial-failure recovery skips checksum-matching
  published versions; it does not republish them or apply to the legacy workflow.

## Rollout tracker

Snapshot: **2026-11-22**, read-only main trees/manifests/workflows, GitHub release
lists, crates.io and PyPI APIs. No settings changed, adoption dry runs executed,
or credentials inspected. `trustpub_only` below is the literal
**`crate.trustpub_only`** field from crates.io, fetched with a User-Agent.
An unpublished crate has no such setting; N/A is not `false`. Matching registry
names alone do not establish ownership: unrelated projects are excluded.
No PyPI package was found for the listed project names except spatial-typestate;
PyPI's `pelorus` belongs to an unrelated project.

Workflow cells list all `.github/workflows` basenames at the linked commit;
`—` means no workflows. **Pending adoption PR** means a PR is needed, not that
one is already open (none of the inspected open PRs implements this standard).
**Adopted** requires a merged adoption and successful PR dry run; no repository
has that evidence yet. **Not applicable (publishes nothing)** means no software
package/binary release in scope now, not permanent exemption from future releases.

| Repository | crates.io / `crate.trustpub_only` (`only`) | PyPI | Current workflows | Adoption |
| --- | --- | --- | --- | --- |
| [.github](https://github.com/TelemetryWorks/.github/tree/686469bc668502637723dd13259cf5b9313bb7f0) | — | — | — | **Not applicable (publishes nothing)** |
| [spatial-typestate](https://github.com/TelemetryWorks/spatial-typestate/tree/4011671426926f3ea31fe5573a73876c613e0c5e) | `spatial-typestate` 0.1.0; 0.0.1 placeholder; only=true | `spatial-typestate` 0.1.0; Rust binding crate is internal (publish=false) | `candidate-qualification.yml`, `ci.yml`, `codeql.yml`, **RETIRED `publish-0.1.0.yml`**, `python.yml`, `scorecard.yml`, `sonar-analysis.yml`, `trusted-publishing.yml`, `zizmor.yml` | **Pending adoption PR**; preserve Python qualifications; retired route prohibited |
| [irig106-validator](https://github.com/TelemetryWorks/irig106-validator/tree/031024e2e3715459c508fed7ac2537ecd651ece3) | No crate | None found | — | **Not applicable (publishes nothing)** |
| [irig106-types](https://github.com/TelemetryWorks/irig106-types/tree/bd8cd4115e1b7ec89edbcf7de03e5a4406d6fd2e) | `irig106-types` 0.1.0; 0.0.1 placeholder; only=true | None found | `ci.yml`, `scorecard.yml`, `trusted-publishing.yml`, `zizmor.yml` | **Pending adoption PR**; older shape, not adopted |
| [irig106-core](https://github.com/TelemetryWorks/irig106-core/tree/38ac0700d40152d96642cb11176f2e0cdd347c53) | `irig106-core` 0.0.1 placeholder; only=true | None found | — | **Adopt before first real release** |
| [irig106-time](https://github.com/TelemetryWorks/irig106-time/tree/96745982a3860ac3aa9102da502a53272c25cb46) | `irig106-time` through 0.7.0 (prototype); only=true. `irig106-time-cli` 0.0.1 placeholder; only=true; main CLI still publish=false | None found | `ci.yml` | **Adopt before first real release**; S20 rebuilt family preparation |
| [irig106-decode](https://github.com/TelemetryWorks/irig106-decode/tree/3afa7a4750eae0f513688c3fc0e71ade8df42b6a) | `irig106-decode` 0.0.1 placeholder; **only=false** (owner turning it on; blocker until verified true) | None found | — | **Adopt before first real release** |
| [irig106-write](https://github.com/TelemetryWorks/irig106-write/tree/fb535f828dfb6e94d39428976ef8895535a7e075) | `irig106-write` 0.0.1 placeholder; only=true | None found | — | **Adopt before first real release** |
| [irig106-cli](https://github.com/TelemetryWorks/irig106-cli/tree/3dbd4eecc0cddb28b1ee8fe39a552562f81fddbe) | `irig106-cli` 0.0.1 placeholder; only=true | None found | — | **Adopt before first real release** |
| [irig106-index](https://github.com/TelemetryWorks/irig106-index/tree/b7c0fc9d32261c75da6e216c9229ae06a8a5cfcd) | `irig106-index` 0.0.1 placeholder; only=true | None found | — | **Adopt before first real release** |
| [irig106-docs](https://github.com/TelemetryWorks/irig106-docs/tree/76acb7b4545b3cecc4bd31de634ec740e7639807) | No crate | None found | `book.yml` | **Not applicable (publishes nothing)** |
| [irig106-studio](https://github.com/TelemetryWorks/irig106-studio/tree/3d1a5789b19f413dbf53952a2a120afa8f7891f7) | `irig106-studio`, `irig106-studio-core`, `irig106-studio-wasm` unpublished; only=N/A | None found | `ci.yml`, `scorecard.yml`, `zizmor.yml` | **Adopt before first real release** |
| [irig106-ch10-reader](https://github.com/TelemetryWorks/irig106-ch10-reader/tree/2fdc4773ff304169e2942a4e9934e30f32e93dd7) | Crate unpublished; only=N/A. Ships GitHub Windows binary archives through v0.1.2; binary variant required | None found | `windows-ci.yml` | **Pending adoption PR**; binary variant |
| [irig106-tmats](https://github.com/TelemetryWorks/irig106-tmats/tree/3197c685b0b419b2144f4a9dce936ddcfd2c63c5) | `irig106-tmats` 0.0.1/0.0.2 placeholders; only=true. `irig106-tmats-cli` unpublished; only=N/A | None found | `ci.yml`, `scorecard.yml`, `zizmor.yml` | **Adopt before first real release**; family 12 preparation |
| [pelorus](https://github.com/TelemetryWorks/pelorus/tree/810f772e3a9e0b3e70123435516eeeb7bb8d62b2) | No TelemetryWorks crate: manifest name `server` and registry name `pelorus` point to unrelated projects; only=N/A for this repo | No TelemetryWorks package; same-name PyPI project is unrelated | `ci.yml`, `scorecard.yml`, `zizmor.yml` | **Adopt before first real release**; before distributing the server |
| [jseries](https://github.com/TelemetryWorks/jseries/tree/35f2aaac9098224366303bf3089b74677597945f) | `jseries-core`, `jseries-schema`, `jseries-cli`, `jseries-python` unpublished and workspace publish=false; only=N/A | `jseries` package source exists; unpublished | `ci.yml`, `codeql.yml`, `performance.yml`, `python.yml`, `scorecard.yml`, `sonarcloud.yml`, `zizmor.yml` | **Adopt before first real release**; before enabling any crate/Python upload |
| [rcc-106-standards](https://github.com/TelemetryWorks/rcc-106-standards/tree/10719995d63426c2fe62d1548acb21ae0238b44c) | No crate; mirrored standards assets are outside software-release scope | None found | `audit-mirror.yml`, `validate.yml` | **Not applicable (publishes nothing)** |

Registry evidence: [crates.io API](https://crates.io/api/v1/crates/irig106-types)
uses `/api/v1/crates/<crate>` for each name; [PyPI JSON](https://pypi.org/pypi/spatial-typestate/json)
uses `/pypi/<project>/json`. An HTTP 404 records no published package at that
name, not a completed Trusted Publisher setup. The linked repository snapshots
supply package identities and all workflow filenames. The time and tmats CLI
checks used their **separate** crate endpoints, not the library's settings.
For each adopting row, the registry registration must name exactly
`TelemetryWorks/<repository in that row>` and `trusted-publishing.yml`, with
`crates-io` for every family crate being uploaded and `pypi` for each Python
project. Complete the environment and only-mode checklist for that repository;
a binary-only release has no registry registration until it uploads a package.
No private registry publisher/environment settings were inspected: the owner
must complete the checklist above, even where only-mode is already true.

The tracker distinguishes current publication from future adoption. In particular,
`irig106-time` has historical library releases; S20 is preparation for the rebuilt
family's next authorized release, not a claim that no version has ever shipped.
`pelorus` and `jseries` have application/package sources but no owned registry
publication or binary release yet; adopt before enabling distribution.
`rcc-106-standards` mirrors documents, not TelemetryWorks software packages.

Each repository adoption is **ONE PR**, and its **`pull_request` dry run must
pass before merge**. No real release is triggered by adopting the workflow.
The first real use is the **next authorized release**, with no re-tagging,
replacement assets or re-release of an existing version. Repositories without
a real release adopt during release preparation (time S20; tmats family 12).
Update this tracker with the adoption PR and dry-run evidence after review and
merge; do not change a row to Adopted merely because the filename exists.
