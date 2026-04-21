# Pain Track Documentation

This repository is a documentation hub for the Pain Track ecosystem. It does not contain the application source code itself. Instead, it consolidates the material needed to understand the current and legacy Pain Track systems, their architecture, technical stack, user flows, supporting integrations, and the highest-priority technical and security risks.

The repository is now organized around five canonical documents.

## Purpose

This repository exists to help technical and non-technical readers answer a few core questions:

- What is the difference between the old and current Pain Track systems?
- Which repositories and production services are still active?
- How do the main flows work from a user and system perspective?
- How is the data model organized?
- Which integrations exist outside the core frontend and backend?
- What are the main maintainability and security risks?

In practice, this repository supports system review, onboarding, architecture discovery, risk assessment, and planning for future improvements or retirement decisions.

## Canonical Documents

Read these files as the main documentation set:

- [01_architecture_and_repositories.md](01_architecture_and_repositories.md): repository landscape, runtime architecture, data model, and relationship behavior
- [02_technical_stack_and_infrastructure.md](02_technical_stack_and_infrastructure.md): hosting, deployment model, environments, integrations, and configuration surfaces
- [03_functional_user_flows.md](03_functional_user_flows.md): main login, episode, track, segment, guest, and discussion flows
- [04_reconciliation_report.md](04_reconciliation_report.md): old documentation versus the current documented reality in this repository
- [05_risks_and_legacy_assessment.md](05_risks_and_legacy_assessment.md): maintainability, testing, legacy, and security risks

## Repository Contents

- [01_architecture_and_repositories.md](01_architecture_and_repositories.md): Main architecture and data-model reference
- [02_technical_stack_and_infrastructure.md](02_technical_stack_and_infrastructure.md): Technical stack and infrastructure reference
- [03_functional_user_flows.md](03_functional_user_flows.md): User-facing flow documentation
- [04_reconciliation_report.md](04_reconciliation_report.md): Historical reconciliation between the old documentation and the current repository narrative
- [05_risks_and_legacy_assessment.md](05_risks_and_legacy_assessment.md): Consolidated risk and legacy assessment
- [aws_ses.md](aws_ses.md): Notes on the AWS SES email-template repository used by the current Pain Track backend
- [server_security_review.md](server_security_review.md): Standalone security review with detailed findings and remediation guidance
- [assets/](assets/): Screenshots referenced by the flow documentation

## What This Repo Explains

Across the available Markdown files, this repository documents five main areas:

1. System landscape: what the old and current Pain Track systems are, where they run, and how they relate.
2. Product behavior: how users log in, create episodes, add tracks, and add segments.
3. Data structure: how users, patients, episodes, tracks, segments, discussions, bookmarks, and auth-related records relate to each other.
4. Integrations: how supporting services such as AWS SES fit into the platform.
5. Risk profile: where the major operational, maintainability, and security concerns currently sit.

## Scope Notes

- This repository is documentation-focused.
- It summarizes both the legacy and active versions of Pain Track.
- It includes both technical and plain-language explanations.
- It captures known gaps, including missing tests, incomplete documentation, and confirmed security concerns.

## Repo Summary

The purpose of this repository is to serve as a single place where someone can understand how Pain Track works, what is still in production, how the core domain is structured, and which areas need attention before future development, migration, or shutdown decisions are made.

## Ecosystem Repositories

The following repositories compose the Pain-Track platform:

* **[pain-app-server](https://github.com/Center-for-Welfare-Metrics/pain-app-server):** The primary Node.js/TypeScript Backend API.
* **[pain-app-frontend](https://github.com/Herikle/pain-app-client):** The main user interface.
* **[legacy-backend-old](https://github.com/Center-for-Welfare-Metrics/old-pain-app-server):** Deprecated backend version (reference only).
* **[legacy-frontend-old](https://github.com/Center-for-Welfare-Metrics/OLD-pain-track-client):** Deprecated frontend version (reference only).
* **[aws_ses](https://github.com/Center-for-Welfare-Metrics/SesPainTrack): Repository for AWS SES email templates used by the current backend.

---

## Guidelines for Contributors
* **Language:** All documentation must be written in **English**.
* **Format:** Documents should use **Markdown (.md)** to ensure version control compatibility and ease of AI processing.
* **Assets:** Any diagrams or screenshots should be stored in an `/assets` folder and linked within the relevant document.

---

## Contact & Maintenance
**Organization:** Center for Welfare Metrics  
**Project Lead:** Wladimir J. Alonso  
**Status:** Phase 1 - Technical Mapping (In Progress)