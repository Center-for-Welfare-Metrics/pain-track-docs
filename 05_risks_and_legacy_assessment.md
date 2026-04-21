# Risks and Legacy Assessment

This document consolidates the main maintainability, testing, operational, and security risks described in this repository.

## Legacy System Assessment

The old Pain Track system should be treated as a legacy system kept online mainly so users can access historical information.

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

The source documents explicitly warn that important historical information could be lost if the old system is shut down before the MongoDB data is reviewed.

## Current System Assessment

The current Pain Track system is the active platform, but the repository documents several quality and operational concerns.

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
10. no automated test gate before main-branch changes

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
- concentrated deployment ownership

## Recommended Next Steps

1. map and review the old MongoDB data before any retirement decision
2. document environment variables and environment setup more rigorously
3. add CI checks and branch protection before allowing changes into `main`
4. add end-to-end coverage for the main flows in the current system
5. remediate the documented authorization and auth-token issues before feature work