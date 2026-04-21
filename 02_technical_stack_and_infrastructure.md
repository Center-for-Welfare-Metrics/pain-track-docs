# Technical Stack and Infrastructure

This document summarizes the technologies, environments, deployment paths, integrations, and configuration surfaces described in this repository.

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
- hosting: Vercel for frontend, Google Cloud Run for backend
- database: MongoDB Atlas
- email service: AWS SES
- AI dependency: OpenAI via configured backend environment variables
- captcha and OAuth integrations: Google services

## Public and Environment URLs

### Old system

- public frontend: `https://cp.pain-track.org`
- public backend: `https://oldpaintrackserver-840926881618.us-central1.run.app`
- calculus service: `https://calculus-840926881618.us-central1.run.app`

### Current system

- public frontend: `https://www.pain-track.org/`
- production backend: `https://newpainappserver-840926881618.us-central1.run.app`
- development backend: `https://painappserverdev-840926881618.us-central1.run.app`
- development branch: `develop`

## Deployment Model

Current understanding for both generations:

1. Code is pushed to GitHub.
2. Frontend deployment is tied to Herikle-controlled GitHub ownership because Vercel is using the free plan.
3. Backend deployment happens through Google Cloud Run when changes reach `main`.

Important limitations:

- there is no GitHub Actions setup for either the client or the server
- there is no test pipeline before changes reach `main`
- frontend deployment is operationally centralized around one person

Recommended improvement already identified in the source documents:

- add CI checks and branch protection so tests run before merge or deployment

## Environment Separation

### Old system

The old system appears to be treated as a production-like legacy environment kept online mainly for historical access.

### Current system

The current system documents separate production and development environments.

Stated rule:

- any non-production URL uses the development backend and the development database

Why this matters:

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

The legacy system depends on an additional service used for episode-related calculations.

- URL: `https://calculus-840926881618.us-central1.run.app`
- language note: Python
- repository: unknown

Operational implication:

- if this service changes or disappears, old-system episode calculations may fail or return incorrect results

### AWS SES

The current app sends email through AWS SES.

The supporting template repository is documented in [aws_ses.md](aws_ses.md).

What the template repository does:

- stores HTML and text email templates
- pushes template updates to AWS SES with update scripts
- does not send the emails itself

Pain Track email flows currently rely on four documented templates:

- contact form
- recovery password
- request email change code
- set password

Relevant credentials:

- `AWS_ID`
- `AWS_SECRET_KEY`

### Google integrations

The documentation references Google services for:

- OAuth login
- reCAPTCHA
- project configuration

### OpenAI integration

The current backend is configured to use OpenAI through environment variables and a configurable model identifier.

Known operational issue:

- the configured model no longer exists, which currently breaks model-dependent backend routes

## Codebase Technology Notes

### Frontend stack

Both frontend generations are based on ReactJS with NextJS and use TypeScript.

The important architectural difference on the frontend is query handling.

#### Current frontend query handling

The current frontend uses React Query for server-state management.

Why this is better:

- loading, error, and success states are handled more consistently
- caching and refetch behavior are centralized
- stale data handling is more explicit
- repeated request logic is less likely to be duplicated across screens

#### Old frontend query handling

The old frontend handles queries in a more manual way.

Why that is worse:

- request logic is more likely to be repeated in multiple places
- loading and error states tend to be handled inconsistently
- caching behavior is weaker or implicit instead of being managed centrally
- invalidation and refresh behavior is harder to reason about
- the UI becomes more tightly coupled to fetch timing and component-level state

In practice, this raises maintenance cost and makes regressions in data-loading flows more likely.

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
- codebase appears more organized than the old system
- still needs dependency updates and better test coverage

## Infrastructure Posture

### Local development setup

Each repository, both frontend and backend, has README instructions explaining how to run locally.

So local startup guidance exists and the basic workflow is documented.

### Operational maturity limits

The main issue is not lack of explanation. The setup is understandable, but the supporting operational controls are still weak and, in several areas, absent.

The following capabilities do not currently exist in the documented setup:

- CI pipeline
- branch protection rules
- observability or logging strategy
- secret rotation procedures
- backup and recovery procedures
- rate limiting or abuse controls at infrastructure level

## Operational Summary

Pain Track currently runs as a GitHub-backed, Vercel-and-Cloud-Run application stack on top of MongoDB Atlas, with additional dependencies on AWS SES, Google services, and OpenAI. The setup is documented well enough to understand how it runs, but its operational posture is still weak because key controls such as CI, branch protection, observability, secret rotation, backups, and abuse protections are not in place.