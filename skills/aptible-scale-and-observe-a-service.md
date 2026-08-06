---
name: Scale an Aptible service and watch the operation
description: Change container count or size for a service, then follow the resulting operation to completion and confirm the new release.
api: openapi/aptible-deploy-openapi-original.yml
generated: '2026-08-06'
method: generated
source: openapi/aptible-deploy-openapi-original.yml + conventions/aptible-conventions.yml
operations:
  - GetAppByHandle
  - ListServicesForApp
  - CreateOperationForService
  - GetOperation
  - ListReleasesForService
  - ListOperationsForService
  - ListServiceSizingPoliciesForService
---

# Scale an Aptible service and watch the operation

## Steps

1. **Find the app.** `GetAppByHandle` (`GET /find/app`) — pass the account handle
   too, since app handles are unique only within an environment.
2. **List its services.** `ListServicesForApp` (`GET /apps/{app_id}/services`).
   A service is one process type (`web`, `worker`, …) from the Procfile.
3. **Scale.** `CreateOperationForService`
   (`POST /services/{service_id}/operations`) with `{"type": "scale"}` plus
   `container_count` and/or `container_size` / `instance_profile`.
4. **Poll** `GetOperation` (`GET /operations/{id}`) until `succeeded`.
5. **Confirm.** `ListReleasesForService` (`GET /services/{service_id}/releases`)
   shows the new release; `ListOperationsForService` is the audit trail.

## Autoscaling

`ListServiceSizingPoliciesForService`
(`GET /services/{service_id}/service_sizing_policies`) reads the autoscaling
policy; `CreateServiceSizingPolicy` and `UpdateServiceSizingPolicy` change it.
Prefer setting a policy over repeatedly issuing manual scale operations.

## Rules

- Scaling **costs money and restarts containers**. It is a write with real
  consequence — see `agentic-access/aptible-agentic-access.yml`.
- A scale operation can be cancelled mid-flight with `PatchOperation`
  (`PATCH /operations/{id}`).
- No idempotency key exists. A repeated scale POST enqueues a second operation.
