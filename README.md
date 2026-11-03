# Tantusz API Specs

This repository contains the API specifications for the Tantusz project in OpenAPI 3.0 (Swagger) format.

## Directory Structure and Contents

- `services/bank-service.yaml`: OpenAPI 3.0.3 specification for Google sign-in, browser sessions, and user-owned Open Banking integration.
- `components/components.yaml`: API components and reusable schemas (`schemas`).
- `services/assistant-service.yaml`: public, session-authenticated assistant conversations and analysis runs.
- `internal/analytics-service.yaml`: read-only internal financial analytics, authenticated with delegated bearer grants.
- `components/assistant.yaml`, `components/analytics.yaml`: shared assistant and exact-decimal analytics schemas.
- `consumers/backend.yaml`, `consumers/assistant.yaml`: generation entry points assembled from the shared contracts, with consumer-specific authentication.

## Browser authentication

Use the same-origin `/api` base URL. Navigate the browser to
`/api/oauth2/authorization/google` to sign in; this endpoint returns a redirect and
must not be called through fetch. The backend stores the authenticated session in
an HttpOnly `SESSION` cookie. `GET /auth/me` returns the signed-in profile or 401.

Fetch `GET /auth/csrf` before POST/DELETE requests, then send the returned `token`
using the returned `headerName`. Fetch a fresh token after sign-in and logout.
`POST /auth/logout` clears the session and returns 204.

Bank connections, accounts, transactions, synchronization, and deletion are scoped
to the signed-in user. Resource IDs belonging to another user return 404. Existing
ownerless records are retained but inaccessible; each user must connect their bank
again. The bank callback requires a single-use `state` and redirects to the frontend
with success or failure status. It does not return a connection JSON body.

## Shared specification workflow

The backend, frontend and assistant consume this repository through their
`swagger/tantusz-api-specs` Git submodules. Commit changes here first, then update
both submodule references and regenerate each consumer. The backend wrapper only
references controller operations; Google authorization and logout are implemented
by Spring Security filters. No generated source should be edited manually.

All API changes are authored only in this repository, never in consumer submodule working trees. The frontend generates only `services/`; `internal/` and `consumers/` are server-side contracts. Consumer builds generate directly from the shared `consumers/` entry points.
