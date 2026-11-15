# TelemetryWorks security program

Status date: 2026-11-15. This is the organization-level controls inventory
and policy. See [SECURITY.md](SECURITY.md) for vulnerability reporting and
response commitments, and the [disconnected-network report outline](docs/cybersecurity/disconnected-networks.md)
for evidence still needed for offline deployment claims.

## Evidence and scope

The owner assignment of 2026-11-15 reports GitHub secret scanning, push
protection, Dependabot alerts and security updates, private vulnerability
reporting enabled on all 17 public repositories on that date. The owner’s
review correction records CodeQL default setup configured with languages on
13 repositories, advanced `codeql.yml` on `jseries` and `spatial-typestate`,
and default setup configured with no analysable languages on `.github` and
`irig106-validator` (nothing analyzed in those two repositories).
Those control and CodeQL statements remain owner-reported, rather than an
independent audit, a record of successful scans, or a claim of zero findings.
Repository names were obtained from the public organization repository listing
on 2026-11-15. The follow-up inventory below reads live GitHub protection and
merge settings and public crates.io metadata on 2026-11-15. The revision also
checks main-branch workflow paths in the seven zero-check repositories, and
workflow paths, release instructions, PR #15 and release attempt logs in
`irig106-types`. Links to live endpoints may change.
TODO: archive owner-approved settings snapshots and scan evidence.

Policy requirements below do not establish implementation in any repository.
The release controls marked planned have no release-specific evidence in this
repository. Update each status only with dated, reviewable evidence.

## Publishing policy

Owner policy dated 2026-11-13: crates.io releases must use Trusted Publishing
from GitHub Actions. Enable crates.io Trusted-Publishing-only mode for each
published crate; do not retain a reusable publishing token as an alternative.

Owner decision 2026-11-15: **One-time token bootstrap** is the sole exception
to the 2026-11-13 policy, for a brand-new crate's first upload only. Use a
crate-scoped `publish-new` token with a 1-day expiry. After that upload, revoke
the bootstrap token, configure the Trusted Publisher, and enable
Trusted-Publishing-only mode before any subsequent upload. This exception
does not permit token-based updates to an existing crate.

### Publishing workflow standard and pending migration

Owner decision 2026-11-15: every publishing workflow must use the path
`.github/workflows/trusted-publishing.yml`. Registry Trusted Publishers must
point at the filename `trusted-publishing.yml`; publishing environments are
`crates-io` for crates.io and `pypi` for PyPI. This is the owner standard,
not a claim that every repository has migrated.

Current `irig106-types` main has only `ci.yml` and `release.yml` in
[`.github/workflows`](https://github.com/TelemetryWorks/irig106-types/tree/1eea6ea97f739d429f86529e22aef8b376aec450/.github/workflows).
Publishing still uses `.github/workflows/release.yml`; that workflow published
0.1.0. The owner tracks the pending rename in [types PR #15](https://github.com/TelemetryWorks/irig106-types/pull/15).
The rename is not yet on main, and the inspected PR diff still uses
`release.yml`. **Before the next tag**, the owner must coordinate the workflow
rename and update the crates.io Trusted Publisher to `trusted-publishing.yml`.
With `trustpub_only=true`, a stale publisher filename blocks the release;
a fallback publishing token is not a remedy.

### Verified release and attempt history

Verified release evidence: [crates.io version metadata](https://crates.io/api/v1/crates/irig106-types/0.1.0)
for `irig106-types` 0.1.0 records `trustpub_data.provider=github`, repository
`TelemetryWorks/irig106-types`, run [`43727921675`](https://github.com/TelemetryWorks/irig106-types/actions/runs/43727921675),
and SHA `3c517358694646652939c236968b202ebf76e28a` (`3c51735`).
[Crate metadata](https://crates.io/api/v1/crates/irig106-types) records
`trustpub_only=true` at inspection. This establishes Trusted Publishing for
that version and the current only-mode setting, not every crate or release.
The run's currently displayed conclusion is **failure** (attempt 3), which must
not be confused with the successful publishing attempt. All times below are
UTC on 2026-11-15; each attempt used `.github/workflows/release.yml` at the
SHA above.

| Attempt | Started | Result and evidence |
| --- | --- | --- |
| [1](https://github.com/TelemetryWorks/irig106-types/actions/runs/43727921675/attempts/1) | 15:05:33 | Failed safely before the crates.io Trusted Publisher existed: the authentication step reported no Trusted Publishing configuration for the repository; no upload occurred. |
| [2 — publishing attempt](https://github.com/TelemetryWorks/irig106-types/actions/runs/43727921675/attempts/2) | 15:20:29 | Succeeded; logs record publication of 0.1.0. Crates.io records creation at 15:21:39Z (15:21:39.926580Z). |
| [3](https://github.com/TelemetryWorks/irig106-types/actions/runs/43727921675/attempts/3) | 15:29:13 | Later rerun after the successful upload; failed safely with `crate irig106-types@0.1.0 already exists` on the crates.io index. It did not publish a replacement. |

**Lesson:** never rerun a release workflow after a successful upload. Check
crates.io first, including when a workflow reports failure: the upload may
already have completed. The existing [RELEASING instructions](https://github.com/TelemetryWorks/irig106-types/blob/1eea6ea97f739d429f86529e22aef8b376aec450/docs/RELEASING.md#package-contents-and-failures)
already direct maintainers to inspect crates.io before treating an
existing-version error as retryable, and prohibit overwriting a version.

TODO: per-crate publisher configuration and any bootstrap completion evidence
without secrets; other crates remain unverified here.

## Main branch policy

Main rulesets must require pull requests and required checks, prohibit force
pushes and branch deletion, and use merge commits for integration. The check
names must match the repository's applicable workflows.

Live GitHub API evidence on 2026-11-15 confirms 16 active `main protection`
rulesets targeting the default branch. They require PRs, block force pushes
and deletion, and restrict PR merges to merge commits (`allowed_merge_methods`
is `["merge"]`). Each has an always-on administrator bypass
(`RepositoryRole`, actor ID `5`, mode `always`): administrators can bypass
these rules. Required checks are configured on nine of
those rulesets. The seven with **no required checks** are `.github`,
`irig106-cli`, `irig106-core`, `irig106-decode`, `irig106-index`,
`irig106-write` and `irig106-validator`. Main-branch tree inspection confirms
that all seven lack a `.github/workflows` directory: **no CI yet** means no
repository-defined GitHub Actions workflows on main in this inventory; it
does not rule out external checks. Per-repository tree evidence is linked
below. No required-check gate is enforced there.

`spatial-typestate` uses [classic main branch protection](https://api.github.com/repos/TelemetryWorks/spatial-typestate/branches/main/protection):
PRs and 27 status checks are required, force pushes/deletion are disabled,
and `enforce_admins=true`. Unlike the 16 rulesets, this protection does not
limit integration to merge commits: [repository merge settings](https://api.github.com/repos/TelemetryWorks/spatial-typestate)
still allow merge, squash and rebase. This is an exception to the merge-commit
policy above. On the 16 ruleset repositories, repository settings also allow
all three methods, but the ruleset restricts non-bypassing PR merges.

Per-repository links and required-check counts below cite the inspected
protection endpoints, including their configured check names. A configured
check is not evidence that it passed; no bypass was used or settings changed.

## Controls inventory

| Control | Status at 2026-11-15 | Evidence / remaining work |
| --- | --- | --- |
| GitHub secret scanning and push protection | Enabled on all 17 public repositories, owner-reported | TODO: dated settings evidence and alert handling records |
| Dependabot alerts and security updates | Enabled on all 17 public repositories, owner-reported | TODO: settings evidence; applicability depends on supported dependency manifests |
| Private vulnerability reporting | Enabled on all 17 public repositories, owner-reported | [Reporting procedure](SECURITY.md#reporting-a-vulnerability); TODO: settings evidence |
| CodeQL | Owner-reported: default setup configured with languages on 13; advanced `codeql.yml` on 2; default setup configured with no analysable languages on 2 (nothing analyzed) | See per-repository status below; TODO: language lists, successful analysis runs and triage evidence; configuration alone does not establish coverage |
| Main protection and merge commits | 16 active rulesets with admin bypass; classic protection on `spatial-typestate` | Live evidence below; seven rulesets have no required checks; `spatial-typestate` still permits squash/rebase |
| Trusted Publishing and Trusted-Publishing-only mode | Required policy; verified for `irig106-types` 0.1.0 and current `trustpub_only=true` | Crates.io evidence above; other crates and bootstrap completion evidence remain TODO |
| SLSA build provenance via GitHub artifact attestations | Planned for released binaries | TODO: workflows, release-bound attestations and verification results; no SLSA level claimed |
| Software bills of materials (SBOMs) | Planned for released binaries | TODO: release-bound SBOMs, format, dependency completeness and validation |
| OpenSSF Scorecard | Planned | TODO: workflow, results and remediation tracking |
| OpenSSF Best Practices badge | Planned | TODO: project registration and evidence for badge criteria; no badge claimed |

## Per-repository status

**Enabled** in the other GitHub controls column means owner-reported
secret scanning/push protection, Dependabot alerts, Dependabot security updates
and private vulnerability reporting. CodeQL is recorded separately per repository:
configuration does not establish a successful scan or absence of findings.
The language names and successful analysis evidence remain TODO.
Main protection counts are configured required-check counts, not successful
runs. **Ruleset** rows share the PR, merge-commit, force-push/deletion and
admin-bypass settings described above. **Policy / TODO** means required
where applicable, with per-crate evidence still to collect.
**Planned*** means provenance and SBOMs are planned for released binaries;
artifact applicability must first be established for each repository.

| Repository | Other GitHub controls | CodeQL configuration (owner-reported) | Main protection (live evidence) | Trusted Publishing | Provenance / SBOMs | Scorecard / Best Practices badge |
| --- | --- | --- | --- | --- | --- | --- |
| `.github` | Enabled | Default setup configured; no analysable languages; nothing analyzed | [Active ruleset](https://api.github.com/repos/TelemetryWorks/.github/rulesets/27327754); 0 required checks ([no CI yet](https://api.github.com/repos/TelemetryWorks/.github/git/trees/592197d8330fce45564067f031184132a5e9433d?recursive=1)) | Policy / TODO (if publishing crates) | Planned* | Planned |
| `spatial-typestate` | Enabled | Advanced `codeql.yml` | [Classic protection](https://api.github.com/repos/TelemetryWorks/spatial-typestate/branches/main/protection); 27 checks; squash/rebase allowed | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-validator` | Enabled | Default setup configured; no analysable languages; nothing analyzed | [Active ruleset](https://api.github.com/repos/TelemetryWorks/irig106-validator/rulesets/27327825); 0 required checks ([no CI yet](https://api.github.com/repos/TelemetryWorks/irig106-validator/git/trees/031024e2e3715459c508fed7ac2537ecd651ece3?recursive=1)) | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-types` | Enabled | Default setup configured with languages | [Active ruleset](https://api.github.com/repos/TelemetryWorks/irig106-types/rulesets/27327820); 6 required checks | [0.1.0 via Trusted Publishing](https://crates.io/api/v1/crates/irig106-types/0.1.0) via `release.yml` (attempt 2); `trustpub_only=true`; standard filename migration pending | Planned* | Planned |
| `irig106-core` | Enabled | Default setup configured with languages | [Active ruleset](https://api.github.com/repos/TelemetryWorks/irig106-core/rulesets/27327774); 0 required checks ([no CI yet](https://api.github.com/repos/TelemetryWorks/irig106-core/git/trees/38ac0700d40152d96642cb11176f2e0cdd347c53?recursive=1)) | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-time` | Enabled | Default setup configured with languages | [Active ruleset](https://api.github.com/repos/TelemetryWorks/irig106-time/rulesets/27327804); 12 required checks | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-decode` | Enabled | Default setup configured with languages | [Active ruleset](https://api.github.com/repos/TelemetryWorks/irig106-decode/rulesets/27327779); 0 required checks ([no CI yet](https://api.github.com/repos/TelemetryWorks/irig106-decode/git/trees/3afa7a4750eae0f513688c3fc0e71ade8df42b6a?recursive=1)) | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-write` | Enabled | Default setup configured with languages | [Active ruleset](https://api.github.com/repos/TelemetryWorks/irig106-write/rulesets/27327827); 0 required checks ([no CI yet](https://api.github.com/repos/TelemetryWorks/irig106-write/git/trees/fb535f828dfb6e94d39428976ef8895535a7e075?recursive=1)) | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-cli` | Enabled | Default setup configured with languages | [Active ruleset](https://api.github.com/repos/TelemetryWorks/irig106-cli/rulesets/27327769); 0 required checks ([no CI yet](https://api.github.com/repos/TelemetryWorks/irig106-cli/git/trees/3dbd4eecc0cddb28b1ee8fe39a552562f81fddbe?recursive=1)) | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-index` | Enabled | Default setup configured with languages | [Active ruleset](https://api.github.com/repos/TelemetryWorks/irig106-index/rulesets/27327791); 0 required checks ([no CI yet](https://api.github.com/repos/TelemetryWorks/irig106-index/git/trees/b7c0fc9d32261c75da6e216c9229ae06a8a5cfcd?recursive=1)) | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-docs` | Enabled | Default setup configured with languages | [Active ruleset](https://api.github.com/repos/TelemetryWorks/irig106-docs/rulesets/27327785); 2 required checks | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-studio` | Enabled | Default setup configured with languages | [Active ruleset](https://api.github.com/repos/TelemetryWorks/irig106-studio/rulesets/27327796); 3 required checks | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-ch10-reader` | Enabled | Default setup configured with languages | [Active ruleset](https://api.github.com/repos/TelemetryWorks/irig106-ch10-reader/rulesets/27327759); 2 required checks | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-tmats` | Enabled | Default setup configured with languages | [Active ruleset](https://api.github.com/repos/TelemetryWorks/irig106-tmats/rulesets/27327811); 10 required checks | Policy / TODO (if publishing crates) | Planned* | Planned |
| `pelorus` | Enabled | Default setup configured with languages | [Active ruleset](https://api.github.com/repos/TelemetryWorks/pelorus/rulesets/27327831); 1 required check | Policy / TODO (if publishing crates) | Planned* | Planned |
| `jseries` | Enabled | Advanced `codeql.yml` | [Active ruleset](https://api.github.com/repos/TelemetryWorks/jseries/rulesets/27327829); 11 required checks | Policy / TODO (if publishing crates) | Planned* | Planned |
| `rcc-106-standards` | Enabled | Default setup configured with languages | [Active ruleset](https://api.github.com/repos/TelemetryWorks/rcc-106-standards/rulesets/27327834); 2 required checks | Policy / TODO (if publishing crates) | Planned* | Planned |

## Release evidence and offline use

Release evidence should bind the artifact digest to its source revision,
builder identity, workflow and dependencies. Planned GitHub artifact
attestations supply SLSA provenance; their presence alone is not a security
review, reproducibility proof, or SLSA-level assessment. Planned SBOMs describe
release contents, not an assurance that all vulnerabilities are known.

Offline verification requires preparing the artifact, attestation bundle,
trusted roots and verification tooling before disconnection, with an approved
repository/workflow identity policy. A checksum alone detects corruption only
when the expected checksum is obtained through a trusted channel.
TODO: publish and test a complete offline verification kit for each release.

Technical references: [GitHub artifact attestations](https://docs.github.com/en/actions/concepts/security/artifact-attestations),
[GitHub offline attestation verification](https://docs.github.com/en/actions/how-tos/secure-your-work/use-artifact-attestations/verify-attestations-offline),
and [crates.io Trusted Publishing](https://crates.io/docs/trusted-publishing).

## Maintaining this inventory

Use reviewed documentation changes to record dates, repository or crate scope,
and links to settings evidence, workflow runs or release artifacts. Do not
promote planned controls to implemented based solely on policy text. Existing
crate-specific engineering practice claims remain in [SECURITY.md](SECURITY.md#security-practices)
and need their own evidence when carried into a deployment report.
