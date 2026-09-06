# Risks and Legacy Assessment

This document consolidates the main maintainability, testing, operational, and security risks described in this repository.

> **Current status (2026-09-06):** both generations are retained for preservation and review. The current-generation application is intentionally paused, its backend remains private/hibernated, and no reactivation or new deployment has occurred.

## Legacy System Assessment

The old Pain Track repositories should be treated as legacy source retained for dependency, data-preservation, and historical-behavior review. Current public availability was not established by this documentation task.

### Main legacy risks

1. unknown critical data remains in MongoDB
2. broken environment-dependent integrations exist
3. the backend is untyped JavaScript
4. dependencies are outdated
5. tests are missing or insufficient
6. frontend styling patterns are mixed
7. silent failures are possible
8. documentation is incomplete
9. deployment traceability is poor
10. external dependencies such as the calculus service increase fragility

### Legacy shutdown caution

The source documents historically warned that important information could be lost if legacy services were shut down before MongoDB data was reviewed. That warning remains a preservation constraint, not evidence that a legacy service is currently online.

## Current System Assessment

The current-generation source is WFI's canonical implementation, but it is not an active platform. The repository documents several quality and operational concerns that must be considered before either reactivation or reuse in a replacement.

### Main current-system risks

1. no end-to-end tests
2. low test coverage
3. outdated dependencies
4. known security vulnerabilities
5. missing detailed documentation
6. regression risk after changes
7. environment inconsistencies between development and production
8. undocumented environment configuration details
9. lack of observability
10. no complete automated test gate before main-branch changes

### Known product and technical issues

- responsiveness issues on some frontend screens
- guest flow bug affecting track and segment field entry after episode creation
- backend error caused by an invalid configured GPT model

## Code Quality And Testing

### Old system

- backend uses pure JavaScript without type safety
- frontend uses TypeScript but still has outdated dependencies and mixed styling approaches

### Current system

- the codebase appears more organized than the old system
- some frontend unit tests exist
- backend likely has no tests
- coverage is still low and does not provide a strong regression safety net

### Why this matters

Without a stronger test strategy, changes can break working flows such as login, episode creation, or segment editing without being detected early.

## Security Review Summary

The most serious security problems described in this repository are authorization failures, guest mutation weaknesses, non-expiring JWTs, plaintext storage of password-reset secrets, and missing abuse controls.

The detailed findings, affected files, and remediation order are documented in [server_security_review.md](server_security_review.md).

## Risk Themes Across The Repository

The recurring themes across both systems are:

- under-documented operational setup
- fragile ownership and access control rules
- insufficient automated testing
- dependency drift and stale integrations
- unresolved credential and third-party integration custody

## Controls Added During Institutionalization

- WFI now owns the four current-generation repositories.
- The server `main` branch requires a pull request and one approval.
- The three former Cloud Build triggers are disabled, preventing an ordinary merge from automatically deploying the hibernated backend.
- The GCP project is under WFI organization and billing governance.

These controls preserve ownership and hibernation. They do not remediate dependency or source-level security findings, create a verified logical database backup, retire old credentials, or make the application safe to reactivate.

## Recommended Next Steps

1. execute a separately approved institutional backup and isolated restoration verification before access cleanup
2. retire or rotate credentials only after dependencies and shared-resource effects are verified
3. keep the system hibernated until WFI chooses between rehabilitation and a compatible replacement
4. if reactivation is chosen, add CI and end-to-end coverage for the preserved critical flows
5. remediate the documented authorization, auth-token, and dependency issues before exposing a backend
