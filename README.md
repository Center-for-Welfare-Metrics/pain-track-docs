# Pain Track Documentation

This public repository is the canonical technical documentation hub for the current Pain Track source and its retained legacy implementations. It does not contain the application source code and it is not an authority for Welfare Footprint Framework methodology.

## Current status

**Pain Track is intentionally paused.** WFI has preserved the application source and institutional cloud project for possible reactivation or replacement, but has not reactivated the application, created a new deployment, or exposed a public backend.

- The canonical current frontend source is the private WFI repository [`welfare-footprint-institute/pain-app-client`](https://github.com/welfare-footprint-institute/pain-app-client).
- The canonical current backend source is [`welfare-footprint-institute/pain-app-server`](https://github.com/welfare-footprint-institute/pain-app-server).
- The retained email-template source is [`welfare-footprint-institute/SesPainTrack`](https://github.com/welfare-footprint-institute/SesPainTrack).
- The existing site at `https://www.pain-track.org/` is a continuity deployment in a Herikle-controlled Vercel project. It is not the institutional deployment target and must not be treated as proof that the WFI source is deployed.
- The GCP project that preserves the hibernated backend resources is institutionally governed and billed by WFI. Its Cloud Run services remain private/disabled for public access, and the former automatic Cloud Build triggers are disabled.
- Dependency remediation, credential retirement, an institutional logical backup, and any future runtime reconstruction remain outstanding and separately controlled work.

Documentation below describes historical application behavior unless it explicitly says that a capability is currently available. Login, registration, recovery, contact, AI, and other backend-dependent flows should not be assumed operational while the system is paused.

## Purpose

This repository preserves the material needed to understand:

- the current and legacy repository landscape;
- the historical runtime architecture and application flows;
- the MongoDB/Mongoose data model and scientific-calculation dependencies;
- the supporting integrations and their reactivation implications;
- the known maintainability and security risks that remain before any future deployment.

## Canonical documents

- [01_architecture_and_repositories.md](01_architecture_and_repositories.md): repository landscape, historical runtime architecture, data model, and relationship behavior
- [02_technical_stack_and_infrastructure.md](02_technical_stack_and_infrastructure.md): stack, environments, historical deployment paths, integrations, and current pause state
- [03_functional_user_flows.md](03_functional_user_flows.md): preserved description of historical login, episode, track, segment, guest, and discussion flows
- [04_reconciliation_report.md](04_reconciliation_report.md): relationship between the legacy and later implementations
- [05_risks_and_legacy_assessment.md](05_risks_and_legacy_assessment.md): maintainability, testing, dependency, legacy, and security risks
- [aws_ses.md](aws_ses.md): retained AWS SES template component and its current dormant status
- [server_security_review.md](server_security_review.md): source-level security findings that must be reconsidered before reactivation
- [assets/](assets/): screenshots illustrating the historically implemented flows

## Repository landscape

### Current WFI-owned repositories

- [`pain-app-client`](https://github.com/welfare-footprint-institute/pain-app-client) — canonical current frontend source; private; paused
- [`pain-app-server`](https://github.com/welfare-footprint-institute/pain-app-server) — canonical current backend source; public source, with no public runtime implied
- [`pain-track-docs`](https://github.com/welfare-footprint-institute/pain-track-docs) — this public technical documentation hub
- [`SesPainTrack`](https://github.com/welfare-footprint-institute/SesPainTrack) — retained SES template source; not evidence of a currently usable WFI email service

### Excluded legacy repositories

- [`Center-for-Welfare-Metrics/old-pain-app-server`](https://github.com/Center-for-Welfare-Metrics/old-pain-app-server) — legacy backend retained for dependency and data-preservation review
- [`Center-for-Welfare-Metrics/OLD-pain-track-client`](https://github.com/Center-for-Welfare-Metrics/OLD-pain-track-client) — legacy frontend retained for dependency and historical-behavior review

The two legacy repositories were not part of the WFI ownership transfer and are not declared retired or safe to delete.

## Scope and evidence boundary

- Architecture, data-model, and user-flow descriptions preserve the behavior represented by the source at the time of review; they are not service-availability claims.
- Provider-side ownership, access, billing, backup, and recovery evidence is maintained in WFI's restricted institutional continuity record rather than this public repository.
- No credentials, connection strings, database contents, or token-bearing provider links belong in this repository.

## Contributor guidance

- Write technical documentation in English and Markdown.
- Put diagrams and screenshots in `assets/` and link them from the relevant document.
- Treat reactivation, deployment, dependency modernization, credential changes, and provider configuration as separately approved work.
- Preserve historical behavior and data-model documentation when adding current-status corrections.

## Contact and maintenance

**Organization:** Welfare Footprint Institute

**Institutional owner:** Wladimir J. Alonso

**Technical maintenance contact:** Moritz Bormann

**Status as of 2026-09-06:** Institutionally preserved and intentionally paused; ownership work is only partially complete across the wider system.
