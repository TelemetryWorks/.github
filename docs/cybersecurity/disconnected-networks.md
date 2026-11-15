# Disconnected-network trust report — outline

**Draft outline only.** This document defines questions and evidence to collect;
it does not establish that any product is suitable for an air-gapped site.
TODO means evidence is not supplied here and must be collected and reviewed
before the final report makes the associated claim.

## 1. Scope, products and versions

Define each assessed product, version, commit, target, build features and deployment
mode. Separate library behavior, executable behavior, build-time downloads and
host/application behavior. An organization-wide statement cannot substitute for
product-specific results.

Evidence: the [organization profile](../../profile/README.md) establishes the
telemetry-tooling mission; the [security program](../../SECURITY-PROGRAM.md)
records policy and owner-reported controls. TODO: versioned product inventory,
release manifests, supported platforms and deployment diagrams. Repository names
below are inventory entries, not proof of a product's behavior.

## 2. Runtime network access — verify per product

Proposed question: can each assessed deployment complete its intended functions
without network access? Verify DNS, sockets, listeners, HTTP, update/license
checks, remote resources, plugins and subprocesses under default and optional
features, startup, normal operation, malformed inputs and failure paths.
Telemetry data processing is distinct from outbound usage reporting.

Evidence: TODO for every row: source/dependency review at a pinned revision,
feature inventory, blocked-network functional tests, traffic/syscall captures,
and a documented test environment. Neither repository names nor the Rust
implementation language establish absence of network behavior.

| Repository / potential product | What must be verified | Evidence status |
| --- | --- | --- |
| `.github` | Establish whether any executable tooling or generated content is shipped; if documentation-only, justify runtime assessment as not applicable | TODO |
| `spatial-typestate` | Identify shipped libraries/executables and all features; test network calls, listeners, subprocesses and embedding behavior for each deployment mode | TODO |
| `irig106-validator` | Identify shipped libraries/executables and all features; test network calls, listeners, subprocesses and embedding behavior for each deployment mode | TODO |
| `irig106-types` | Identify shipped libraries/executables and all features; test network calls, listeners, subprocesses and embedding behavior for each deployment mode | TODO |
| `irig106-core` | Identify shipped libraries/executables and all features; test network calls, listeners, subprocesses and embedding behavior for each deployment mode | TODO |
| `irig106-time` | Identify shipped libraries/executables and all features; test network calls, listeners, subprocesses and embedding behavior for each deployment mode | TODO |
| `irig106-decode` | Identify shipped libraries/executables and all features; test network calls, listeners, subprocesses and embedding behavior for each deployment mode | TODO |
| `irig106-write` | Identify shipped libraries/executables and all features; test network calls, listeners, subprocesses and embedding behavior for each deployment mode | TODO |
| `irig106-cli` | Identify shipped libraries/executables and all features; test network calls, listeners, subprocesses and embedding behavior for each deployment mode | TODO |
| `irig106-index` | Identify shipped libraries/executables and all features; test network calls, listeners, subprocesses and embedding behavior for each deployment mode | TODO |
| `irig106-docs` | Establish whether any executable tooling or generated content is shipped; if documentation-only, justify runtime assessment as not applicable | TODO |
| `irig106-studio` | Identify shipped libraries/executables and all features; test network calls, listeners, subprocesses and embedding behavior for each deployment mode | TODO |
| `irig106-ch10-reader` | Identify shipped libraries/executables and all features; test network calls, listeners, subprocesses and embedding behavior for each deployment mode | TODO |
| `irig106-tmats` | Identify shipped libraries/executables and all features; test network calls, listeners, subprocesses and embedding behavior for each deployment mode | TODO |
| `pelorus` | Identify shipped libraries/executables and all features; test network calls, listeners, subprocesses and embedding behavior for each deployment mode | TODO |
| `jseries` | Identify shipped libraries/executables and all features; test network calls, listeners, subprocesses and embedding behavior for each deployment mode | TODO |
| `rcc-106-standards` | Establish whether any executable tooling or generated content is shipped; if documentation-only, justify runtime assessment as not applicable | TODO |

## 3. No usage telemetry or automatic reporting

Proposed question: do products collect, persist or transmit usage, crash or
analytics reports, identifiers, diagnostic payloads or automatic update requests?
Assess opt-in features, bundled dependencies and user-triggered support exports.
Do not turn an absence of observed traffic into an unconditional no-telemetry claim.

Evidence: TODO: per-product source and dependency review, data-flow inventory,
configuration defaults, diagnostic output review and runtime capture results.
The profile's use of “telemetry” describes the application domain, not an
analytics/privacy guarantee.

## 4. Minimal and reviewable dependency graphs

Define “minimal” with explicit product requirements and measured direct,
transitive, optional, build and native dependencies. Assess build scripts,
feature selections, vendoring and offline dependency availability separately
from runtime networking.

Evidence: [SECURITY.md](../../SECURITY.md#security-practices) documents `cargo deny`
checks in two crates; this is a documented claim, not a new audit.
TODO: pinned manifests/lockfiles, resolved graphs and counts per target/feature,
licence and advisory reports, native dependency inventory, vendored-source
checksums and a successful offline build record. No minimality conclusion yet.

## 5. Artifact integrity, identity and offline verification

Outline an online staging and approved-media transfer process for binaries,
checksums, Sigstore/GitHub attestation bundles, trusted roots, SBOMs and a pinned
verifier. Verify on a disconnected host with explicit expected repository and
workflow identity, artifact digest and provenance predicate. Define how trust
roots and the verifier are authenticated and updated. Explain that a valid
signature does not establish code safety, and a checksum without a trusted
reference does not establish origin.

Evidence: [GitHub's offline verification procedure](https://docs.github.com/en/actions/how-tos/secure-your-work/use-artifact-attestations/verify-attestations-offline)
and [CLI verification options](https://cli.github.com/manual/gh_attestation_verify)
describe mechanisms, not TelemetryWorks release readiness.
TODO: actual release assets, checksums, bundles, trusted roots, documented
identity constraints and an end-to-end test with networking disabled, including
tampered-artifact and wrong-identity rejection. TODO: release-bound SBOMs,
format/schema validation, dependency completeness and inventory comparison.
Provenance and SBOM delivery are [planned](../../SECURITY-PROGRAM.md#controls-inventory).

## 6. Reproducibility goals

Goal: independently rebuild an identified release from approved offline inputs
and compare outputs. Specify pinned toolchains/dependencies, build environment,
flags, timestamps, paths and platform-specific nondeterminism. Explain any
remaining differences rather than claiming reproducible builds prematurely.

Evidence: TODO: reproducible-build instructions, offline input archive,
independent rebuild logs and digest comparisons for each supported target.
An attestation records build claims; it does not prove bit-for-bit reproducibility.

## 7. Vulnerability disclosure and patch delivery to offline sites

Describe an authorized connected liaison for private reporting and advisory
retrieval, approved media transfer, authenticity verification, staging tests,
rollback and installed-version tracking. Establish a cadence for importing
advisories and dependency data; disconnected systems cannot learn newly
published advisories automatically. Never require public disclosure of a
sensitive site's data to reproduce a vulnerability.

Evidence: [SECURITY.md](../../SECURITY.md) supplies private reporting, a seven-day
acknowledgment aim, coordinated disclosure and supported-version policy.
TODO: offline-site reporting route, sanitized reproducer guidance, patch delivery
runbook, signed/attested patch kit, advisory snapshots, rollback test and owner
responsibilities. No offline patch-delivery SLA is established by this outline.

## 8. Threat model, limits and explicit non-claims

Cover malicious or malformed input files, compromised build/release systems,
dependencies, removable media, substitution or rollback of artifacts, stale
vulnerability information and a compromised deployment host. Identify trust
boundaries and operator responsibilities for media handling, least privilege,
local access control, backup and update approval.

Evidence: the [security program](../../SECURITY-PROGRAM.md) records controls and
planned supply-chain work. TODO: reviewed per-product threat models, parser
robustness/fuzzing results, isolation tests, release-verification failure tests
and residual-risk acceptance by the deploying site.

Explicit non-claims: no certification, accreditation, regulatory compliance,
SLSA level, guaranteed absence of vulnerabilities or telemetry, universal
network independence, or fitness for safety-critical use is established here.
Air-gapping does not prevent attacks through files, dependencies, media or a
compromised host. Site approval and independent assessment remain separate.

## 9. Evidence register and publication gate

For each proposed final claim, record product/version, evidence link and digest,
method, date, reviewer, result, limitations and remaining TODOs. A final report
must narrow or omit claims without adequate evidence.

Evidence: TODO: completed register, independent review, and an approved final
report. This outline is not that report.
