---
name: Govern the Aptible LLM Gateway
description: Enrol an organization, set model access policies and spend limits, issue and revoke LLM keys, and read usage and request history.
api: openapi/aptible-deploy-openapi-original.yml
generated: '2026-08-06'
method: generated
source: openapi/aptible-deploy-openapi-original.yml + https://www.aptible.com/docs/llm-gateway
operations:
  - CreateLlmGatewayConfiguration
  - ListLlmGatewayConfigurations
  - UpdateLlmGatewayConfiguration
  - CreateOrUpdateLlmPolicy
  - ListLlmPoliciesForAccount
  - GetLlmPolicy
  - GetLlmPolicyUsage
  - ListLlmKeysForAccount
  - GetLlmKey
  - GetLlmKeyUsage
  - GetLlmKeyRequests
  - RevokeLlmKey
---

# Govern the Aptible LLM Gateway

The LLM Gateway fronts 400+ models behind one compliant API. This skill covers
the *governance* surface — the part exposed on `api.aptible.com`.

## Steps

1. **Enrol.** `CreateLlmGatewayConfiguration`
   (`POST /llm_gateway_configurations`). This one endpoint is explicitly safe to
   repeat: the spec documents a 200 "already enrolled (idempotent)". It is the
   only endpoint in the API with that property — do not generalise it.
2. **Check enrolment.** `ListLlmGatewayConfigurations`. Every other LLM endpoint
   returns **403 "organization not enrolled"** until this exists. If you see a
   403 on any `Llm*` operation, go back to step 1 rather than re-authenticating.
3. **Set model access policy.** `CreateOrUpdateLlmPolicy`
   (`POST /accounts/{account_id}/llm_policies/create_or_update`) — an upsert,
   scoped to the environment.
4. **Set the spend limit.** `UpdateLlmGatewayConfiguration`
   (`PATCH /llm_gateway_configurations/{id}`). A trial configuration rejects this
   with **422 "spend_limit cannot be updated on a trial configuration"**.
5. **Audit.** `ListLlmKeysForAccount`, `GetLlmKeyUsage`
   (`GET /llm_keys/{id}/usage`), `GetLlmKeyRequests`
   (`GET /llm_keys/{id}/requests`), `GetLlmPolicyUsage`.
6. **Revoke a key.** `RevokeLlmKey` (`DELETE /llm_keys/{id}`).

## Rules

- The LLM family is the **only** part of the Deploy API that enumerates specific
  status codes (400/403/404/422). Everywhere else you get the generic `default`
  error. Use these codes; do not assume they exist elsewhere.
- Key revocation is immediate and breaks live traffic — confirm with a human.
- Request history contains prompts. Treat `GetLlmKeyRequests` output as sensitive
  and honour `Prefer: no_sensitive_extras=true` where it applies.
