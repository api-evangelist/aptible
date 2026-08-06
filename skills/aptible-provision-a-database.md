---
name: Provision and connect to an Aptible database
description: Pick a database image, provision a managed database into an environment, wait for the provision operation, and read its credentials safely.
api: openapi/aptible-deploy-openapi-original.yml
generated: '2026-08-06'
method: generated
source: openapi/aptible-deploy-openapi-original.yml + conventions/aptible-conventions.yml
operations:
  - ListDatabaseImages
  - GetAccountByHandle
  - CreateDatabase
  - CreateOperationForDatabase
  - GetOperation
  - GetDatabaseByHandle
  - ListDatabaseCredentialsForDatabase
---

# Provision and connect to an Aptible database

## Steps

1. **List the available engines.** `ListDatabaseImages` (`GET /database_images`).
   Each entry carries `type` (postgresql, mysql, redis, elasticsearch, influxdb,
   rabbitmq, sftp), `version` and the `id` you must pass when creating.
2. **Resolve the environment.** `GetAccountByHandle` (`GET /find/account`).
3. **Create.** `CreateDatabase` (`POST /accounts/{account_id}/databases`) with the
   `handle` and the `database_image_id`.
4. **Provision.** `CreateOperationForDatabase`
   (`POST /databases/{database_id}/operations`) with `{"type": "provision"}` and
   the disk/container sizing.
5. **Poll** `GetOperation` until `succeeded`.
6. **Fetch credentials.** `ListDatabaseCredentialsForDatabase`
   (`GET /databases/{database_id}/database_credentials`) returns
   `connection_url` per credential type. `GetDatabaseByHandle`
   (`GET /find/database`) also carries `connection_url` and `passphrase`.

## Rules

- **Credentials are in the payload.** `database.passphrase` and
  `connection_url` are real secrets returned in normal responses. Send
  `Prefer: no_sensitive_extras=true` on every call where you do not specifically
  need them, and never log the raw body.
- **Backups and replicas are graph edges, not flags:**
  `ListBackupsForDatabase` (`GET /databases/{database_id}/backups`) and
  `ListReplicasForDatabase` (`GET /databases/{database_id}/dependents`).
- **Deprovision is an operation**, `{"type": "deprovision"}` posted to
  `/databases/{database_id}/operations` — there is a `DeleteDatabase` in the spec
  but the platform path is the operation. This is destructive and irreversible
  once the retention window lapses; require human confirmation.
