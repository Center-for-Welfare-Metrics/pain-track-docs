# Technical Stack and Infrastructure

This document summarizes the technologies, environments, deployment paths, integrations, and configuration surfaces described in this repository.

Classification rule used in this document:

- **source-confirmed** statements describe the checked-in application or documentation;
- **provider-reported** statements record dated infrastructure verification and are not inferred from source alone;
- historical runtime descriptions do not imply current availability.

> **Current status (2026-09-06):** Pain Track is intentionally paused. WFI owns the canonical current-generation repositories and the GCP project that retains the backend resources. The backend remains hibernated and private, former Cloud Build triggers are disabled, and no new frontend or backend was deployed.

## Stack Summary

### Old system

- frontend repository: `OLD-pain-track-client`
- backend repository: `old-pain-app-server`
- frontend framework: ReactJS with NextJS
- frontend language note: frontend uses TypeScript
- frontend query handling: data fetching is handled in a more ad hoc way rather than through a dedicated query-state library
- backend runtime: Node.js with Express
- backend language note: backend is pure JavaScript
- API style: REST
- hosting: Vercel for frontend, Google Cloud Run for backend
- database: MongoDB Atlas
- external service: calculus service for episode-related calculations

### Current system

- frontend repository: `pain-app-client`
- backend repository: `pain-app-server`
- frontend framework: ReactJS with NextJS
- frontend language note: frontend uses TypeScript
- frontend query handling: React Query is used to manage server-state queries
- backend runtime: Node.js with Express
- backend language note: backend uses TypeScript
- API style: REST
- historical hosting architecture: Vercel for frontend, Google Cloud Run for backend
- canonical source ownership: Welfare Footprint Institute
- current runtime state: intentionally paused; no public WFI backend
- database: MongoDB Atlas
- historical email integration: AWS SES
- historical AI dependency: OpenAI via configured backend environment variables
- historical captcha and OAuth integrations: Google services

## Public and Environment URLs

### Old system

- historical public frontend: `https://cp.pain-track.org`
- historical backend endpoint: `https://oldpaintrackserver-840926881618.us-central1.run.app`
- historical calculus-service endpoint: `https://calculus-840926881618.us-central1.run.app`

### Current system

- continuity frontend URL: `https://www.pain-track.org/`; the Vercel project is controlled by Herikle and is not the WFI institutional deployment target
- historical production backend identifier: `newpainappserver`
- historical development backend identifier: `painappserverdev`
- historical development branch: `develop`

The retained Cloud Run services are private/disabled for public access. The identifiers above preserve architecture and recovery context; they are not operational endpoint claims.

## Deployment Model

Historical source documentation described GitHub-to-Vercel frontend deployment and GitHub-to-Cloud-Build/Cloud-Run backend deployment. The current control state is different:

1. The four current repositories are owned by WFI.
2. The Herikle-controlled Vercel project remains a separate continuity deployment; it was not transferred and is not evidence that the WFI frontend repository deploys automatically.
3. The three identified Cloud Build triggers were disabled. Merging to `main` does not automatically deploy the hibernated backend.
4. The server `main` branch now requires a pull request and one approving review. Review dismissal is restricted to the designated WFI maintainer, while repository/organization administrators retain GitHub's inherent administrative authority.
5. No new CI test gate, frontend hosting target, or deployment pipeline was created.

The absence of automatic deployment is intentional while the application remains paused.

## Environment Separation

### Old system

The old source describes a production-like legacy environment. Whether any legacy service is still reachable was not established by this documentation task.

### Current-generation source

The source documents separate production and development environments historically.

Stated rule:

- any non-production URL uses the development backend and the development database

Why this will matter if reactivation is approved:

- safer feature validation
- lower risk of mixing test activity with real user data
- lower regression risk before production release

## Environment Variables

### Old system variables

- `MONGO_CONNECTION_URL`: MongoDB connection string
- `APP_SECRET`: application secret used by the backend
- `GOOGLE_ID`: Google OAuth client ID
- `GOOGLE_SECRET`: Google OAuth client secret
- `GOOGLE_SCOPES`: Google OAuth scopes used by the application
- `OAUTH_CALLBACK_URL`: OAuth callback URL
- `CALCULUS`: URL of the calculus service used for episode-related calculations
- `WM_DB`: database used by the old system
- `GITLAB_PERSONAL_TOKEN`: GitLab personal access token
- `GITLAB_PROJECT_ID`: GitLab project ID

Known legacy issues:

- OAuth-related variables are no longer working
- GitLab and Google credential setup is broken or incomplete

### Current system variables

- `MONGO_CONNECTION_URL`: MongoDB connection string
- `APP_SECRET`: application secret used by the backend
- `OPENAI_APIKEY`: OpenAI API key
- `OPENAI_ORGANIZATION`: OpenAI organization identifier
- `GOOGLE_RECAPTCHA_SITE_KEY`: Google reCAPTCHA site key
- `GOOGLE_PROJECT_ID`: Google project identifier
- `GPT_MODEL_TO_USE`: OpenAI model configured for use by the application
- `GOOGLE_OATH_CLIENT_ID`: Google OAuth client ID
- `GOOGLE_OATH_SECRET_ID`: Google OAuth client secret
- `OAUTH_REDIRECT_TO`: OAuth redirect URL
- `RECOVERY_PASSWORD_URL`: password recovery flow URL
- `SUPPORT_URL`: support or contact URL used in the application
- `AWS_ID`: AWS access key ID
- `AWS_SECRET_KEY`: AWS secret access key

Known configuration issue:

- the configured GPT model on the new server no longer exists, so backend routes depending on `GPT_MODEL_TO_USE` currently fail until a valid model is configured

## Data and Persistence Infrastructure

MongoDB Atlas is the documented primary database platform for both the old and current systems.

Current system environments documented in this repository:

- production database
- development database

The domain collections and relationships are described in detail in [01_architecture_and_repositories.md](01_architecture_and_repositories.md).

## External Integrations

### Calculus service

The legacy source depended on an additional service used for episode-related calculations.

- URL: `https://calculus-840926881618.us-central1.run.app`
- language note: Python
- repository: unknown

Operational implication:

- inferred from structure: if this service changes or disappears, old-system episode calculations may fail or return incorrect results

### AWS SES

The retained backend was designed to send email through AWS SES. That historical integration is not evidence of a current WFI-controlled or operational email service.

The supporting template repository is documented in [aws_ses.md](aws_ses.md).

What the template repository does:

- stores HTML and text email templates
- pushes template updates to AWS SES with update scripts
- does not send the emails itself

The source defines four email templates:

- contact form
- recovery password
- request email change code
- set password

Relevant credentials:

- `AWS_ID`
- `AWS_SECRET_KEY`

### Google integrations

The source references Google services for:

- OAuth login
- reCAPTCHA
- project configuration

The attempted OAuth branding update did not persist because Google required additional homepage and privacy-policy fields. OAuth metadata therefore remains unresolved and is not a current availability claim.

### OpenAI integration

The retained backend source is configured to use OpenAI through environment variables and a configurable model identifier.

Issue recorded by the earlier source/configuration review:

- the configured model no longer existed at the time of review, so model-dependent routes would require configuration and testing before reactivation

## Codebase Technology Notes

### Frontend stack

Both frontend generations are based on ReactJS with NextJS and use TypeScript.

The important architectural difference on the frontend is query handling.

#### Current frontend query handling

The current frontend uses React Query for server-state management.

Directly confirmed characteristics:

- loading, error, and success handling is managed through React Query
- caching and refetch behavior is centralized in the query layer
- stale data handling is configured explicitly through the query layer
- server-state request logic is managed through React Query rather than ad hoc component code

#### Old frontend query handling

The old frontend handles queries in a more manual way.

Directly confirmed characteristics:

- request logic is handled more manually across the frontend
- loading and error handling is not managed through a dedicated query-state library
- caching behavior is not centralized through React Query
- invalidation and refresh behavior are handled outside a dedicated query layer
- data-fetch timing remains more coupled to component-level state

### Backend stack

Both backend generations use Node.js with Express.

The main language difference is:

- the old backend is JavaScript
- the new backend is TypeScript

### Old system maintainability profile

- backend is untyped JavaScript running on Node.js with Express
- frontend uses ReactJS with NextJS and TypeScript, but query handling is more ad hoc and the codebase also has mixed styling approaches and outdated dependencies

### Current system maintainability profile

- frontend uses ReactJS with NextJS, TypeScript, and React Query for server-state handling
- backend uses Node.js with Express and TypeScript
- still needs dependency updates and better test coverage

## Infrastructure Posture

### Local development setup

Each repository, both frontend and backend, has README instructions explaining how to run locally.

So local startup guidance exists and the basic workflow is documented.

### Operational maturity limits

The setup is documented, but several controls required for safe reactivation remain absent or incomplete.

Current verified controls and limits:

- the server `main` branch has pull-request and one-review protection;
- the former Cloud Build triggers are disabled, preserving hibernation;
- no complete application CI/test pipeline is documented;
- no approved institutional logical backup has been executed;
- secret retirement/rotation remains outstanding;
- observability, recovery testing, and infrastructure-level abuse controls require review before reactivation.

## Operational Summary

Pain Track is preserved as a GitHub-backed Next.js/Node.js application whose historical architecture used Vercel, Cloud Run, MongoDB Atlas, AWS SES, Google services, and OpenAI. WFI now controls the canonical current-generation source and the hibernated GCP project, but the existing Vercel continuity deployment remains separate. The system is not reactivated: there is no public WFI backend, no automatic backend deployment, and no completed institutional logical backup. Dependency remediation, credential retirement, recovery validation, and integration decisions remain future gates.
