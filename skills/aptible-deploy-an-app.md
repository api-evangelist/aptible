---
name: Deploy an app on Aptible
description: Create an environment-scoped app from a Docker image, set its configuration, deploy it, and wait for the deploy operation to finish.
api: openapi/aptible-deploy-openapi-original.yml
generated: '2026-08-06'
method: generated
source: openapi/aptible-deploy-openapi-original.yml + conventions/aptible-conventions.yml
operations:
  - GetAccountByHandle
  - CreateApp
  - CreateOperationForApp
  - GetOperation
  - GetOperationLogs
  - ListServicesForApp
---

# Deploy an app on Aptible

Base URL `https://api.aptible.com`. Every request needs `Authorization: <token>`
(the token is minted at `POST https://auth.aptible.com/tokens`; the OpenAPI models
it as an apiKey in the `Authorization` header, not as `Bearer`). Responses are
`application/hal+json`.

## Steps

1. **Resolve the environment.** Call `GetAccountByHandle` (`GET /find/account`)
   with the environment handle. Aptible calls environments *accounts* in the API.
   Keep the numeric `id`; every create call is scoped by it.
2. **Create the app.** `CreateApp` (`POST /accounts/{account_id}/apps`) with the
   app `handle`. Handles are unique only within an account, so always carry the
   account id alongside.
3. **Deploy.** `CreateOperationForApp` (`POST /apps/{app_id}/operations`) with a
   body of `{"type": "deploy"}` plus the image reference. **This is the part that
   surprises agents:** Aptible does not deploy via a verb on the app. It creates an
   *operation*, and the response is the operation — not the finished deploy.
4. **Poll.** `GetOperation` (`GET /operations/{id}`) until `status` is `succeeded`
   or `failed`. Do not treat the 200 from step 3 as success.
5. **Read the logs on failure.** `GetOperationLogs`
   (`GET /operations/{operation_id}/logs`).
6. **Confirm the running surface.** `ListServicesForApp`
   (`GET /apps/{app_id}/services`) to see the process types that came up.

## Configuration

Environment variables are also an operation: `CreateOperationForApp` with
`{"type": "configure", "env": {...}}`. `CreateConfigurationForApp` exists in the
spec but the configure operation is what the CLI and the first-party MCP server
use. Setting configuration triggers a restart.

## Rules

- **No idempotency key.** Aptible publishes no `Idempotency-Key` contract. If a
  create call times out, do **not** retry blindly — call `ListOperationsForApp`
  and check whether the operation already exists.
- **Errors** come back as `{"code": <int>, "error": "<slug>", "message": "..."}`.
  Branch on `error`, not on the status code: 192 of 195 operations document only a
  generic `default` error response, so the status code alone tells you little.
  See `errors/aptible-problem-types.yml`.
- **Suppress sensitive fields** you do not need by sending
  `Prefer: no_sensitive_extras=true` — app and database payloads otherwise embed
  credentials, passphrases and private keys.
- **Pagination** is `page` / `per_page`.
