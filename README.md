# pain-track-docs
Technical mapping and architectural documentation for the Pain-Track ecosystem.

# Pain-Track Ecosystem Documentation

## Overview
This repository serves as the central knowledge base and "Source of Truth" for the **Pain-Track** project under the **Center for Welfare Metrics**. 

Due to the fragmented nature of the current system—distributed across multiple repositories including frontend, backend, legacy code, and external mathematical scripts—this documentation provides the necessary mapping to understand, maintain, and evolve the ecosystem.

---

## Documentation Map

To ensure clarity and ease of navigation for both developers and AI-assisted tools, the documentation is organized into the following primary documents:

| File | Purpose |
| :--- | :--- |
| **[01_architecture_and_repositories.md](./01_architecture_and_repositories.md)** | Global overview of the system architecture and the specific roles of each repository. |
| **[02_technical_stack_and_infrastructure.md](./02_technical_stack_and_infrastructure.md)** | Details on communication protocols (API), environment variables, Python integrations, and deployment. |
| **[03_functional_user_flows.md](./03_functional_user_flows.md)** | Step-by-step logic for core features: Authentication, Episode creation, and Segment management. |
| **[04_reconciliation_report.md](./04_reconciliation_report.md)** | A critical audit comparing legacy technical specifications with the current live implementation. |
| **[05_risks_and_legacy_assessment.md](./05_risks_and_legacy_assessment.md)** | Assessment of technical debt, systemic risks, and identification of deprecated code. |

---

## Ecosystem Repositories

The following repositories compose the Pain-Track platform:

* **[pain-app-server](https://github.com/Center-for-Welfare-Metrics/pain-app-server):** The primary Node.js/TypeScript Backend API.
* **[pain-app-frontend](INSERT_URL_HERE):** The main user interface.
* **[legacy-backend-old](INSERT_URL_HERE):** Deprecated backend version (reference only).

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
