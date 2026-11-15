# TelemetryWorks security program

Status date: 2026-11-15. This is the organization-level controls inventory
and policy. See [SECURITY.md](SECURITY.md) for vulnerability reporting and
response commitments, and the [disconnected-network report outline](docs/cybersecurity/disconnected-networks.md)
for evidence still needed for offline deployment claims.

## Evidence and scope

The owner assignment of 2026-11-15 reports GitHub secret scanning, push
protection, Dependabot alerts and security updates, private vulnerability
reporting, and CodeQL enabled on all 17 public repositories on that date.
This inventory records that owner-supplied fact; it is not an independent
settings audit, a record of successful scans, or a claim of zero findings.
Repository names were obtained from the public organization repository listing
on 2026-11-15; other repository contents were not inspected for this document.
TODO: attach a durable owner-approved settings snapshot and scan evidence.

Policy requirements below do not establish implementation in any repository.
The release controls marked planned have no release-specific evidence in this
repository. Update each status only with dated, reviewable evidence.

## Publishing policy

Owner policy dated 2026-11-13: crates.io releases must use Trusted Publishing
from GitHub Actions. Enable crates.io Trusted-Publishing-only mode for each
published crate; do not retain a reusable publishing token as an alternative.

The sole bootstrap exception is a one-time, crate-scoped API token for a
brand-new crate's first upload. After that upload, revoke the bootstrap token,
configure the trusted publisher, and enable Trusted-Publishing-only mode before
any subsequent upload. This exception does not permit token-based updates to
an existing crate. TODO: record per-crate publisher configuration,
Trusted-Publishing-only mode, and bootstrap completion evidence without secrets.

## Main branch policy

Main rulesets must require pull requests and required checks, prohibit force
pushes and branch deletion, and use merge commits for integration. The check
names must match the repository's applicable workflows. TODO: record ruleset
exports, required check names and merge-method configuration per repository;
this document does not assert that those settings were audited or changed.

## Controls inventory

| Control | Status at 2026-11-15 | Evidence / remaining work |
| --- | --- | --- |
| GitHub secret scanning and push protection | Enabled on all 17 public repositories, owner-reported | TODO: dated settings evidence and alert handling records |
| Dependabot alerts and security updates | Enabled on all 17 public repositories, owner-reported | TODO: settings evidence; applicability depends on supported dependency manifests |
| Private vulnerability reporting | Enabled on all 17 public repositories, owner-reported | [Reporting procedure](SECURITY.md#reporting-a-vulnerability); TODO: settings evidence |
| CodeQL | Enabled on all 17 public repositories, owner-reported | TODO: supported-language coverage, successful analysis runs and triage evidence; enablement alone does not establish coverage |
| Main rulesets and merge commits | Required policy | TODO: per-repository enforcement evidence |
| Trusted Publishing and Trusted-Publishing-only mode | Required publishing policy, with first-upload exception above | TODO: per-crate implementation evidence; no claim that every crate is configured |
| SLSA build provenance via GitHub artifact attestations | Planned for released binaries | TODO: workflows, release-bound attestations and verification results; no SLSA level claimed |
| Software bills of materials (SBOMs) | Planned for released binaries | TODO: release-bound SBOMs, format, dependency completeness and validation |
| OpenSSF Scorecard | Planned | TODO: workflow, results and remediation tracking |
| OpenSSF Best Practices badge | Planned | TODO: project registration and evidence for badge criteria; no badge claimed |

## Per-repository status

**Enabled** means the five GitHub control groups in the inventory above are
owner-reported enabled: secret scanning/push protection, Dependabot alerts,
Dependabot security updates, private vulnerability reporting, and CodeQL.
It does not mean a scan ran or found nothing. **Policy / TODO** means required
where applicable, with per-repository or per-crate evidence still to collect.
**Planned*** means provenance and SBOMs are planned for released binaries;
artifact applicability must first be established for each repository.

| Repository | GitHub controls | Main rulesets | Trusted Publishing | Provenance / SBOMs | Scorecard / Best Practices badge |
| --- | --- | --- | --- | --- | --- |
| `.github` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `spatial-typestate` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-validator` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-types` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-core` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-time` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-decode` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-write` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-cli` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-index` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-docs` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-studio` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-ch10-reader` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `irig106-tmats` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `pelorus` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `jseries` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |
| `rcc-106-standards` | Enabled | Policy / TODO | Policy / TODO (if publishing crates) | Planned* | Planned |

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
