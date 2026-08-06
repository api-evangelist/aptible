---
name: Pull Aptible compliance evidence
description: Collect the activity reports, intrusion-detection reports and image code-scan results an auditor asks for, straight from the API.
api: openapi/aptible-deploy-openapi-original.yml
generated: '2026-08-06'
method: generated
source: openapi/aptible-deploy-openapi-original.yml + https://www.aptible.com/docs/core-concepts/security-compliance/overview
operations:
  - ListAccountsForStack
  - ListActivityReportsForAccount
  - GetActivityReport
  - GetActivityReportDownload
  - ListIntrustionDetectionReportsForStack
  - GetIntrustionDetectionReport
  - GetIntrustionDetectionReportPdfDownload
  - GetIntrustionDetectionReportCsvDownload
  - ListCodeScanResultsForApp
  - GetCodeScanResult
---

# Pull Aptible compliance evidence

Aptible sells compliance as the product, so the evidence an auditor wants is
addressable over the API rather than only in the dashboard.

## Steps

1. **Enumerate the estate.** `ListStacks` (`GET /stacks`), then
   `ListAccountsForStack` (`GET /stacks/{stack_id}/accounts`) for the
   environments under each dedicated stack.
2. **Activity reports** (who did what, per environment):
   `ListActivityReportsForAccount`
   (`GET /accounts/{account_id}/activity_reports`) → `GetActivityReport` →
   `GetActivityReportDownload`
   (`GET /activity_reports/{activity_report_id}/download`).
3. **Intrusion detection** (HIDS, per stack):
   `ListIntrustionDetectionReportsForStack`
   (`GET /stacks/{stack_id}/intrusion_detection_reports`) →
   `GetIntrustionDetectionReportPdfDownload` or
   `GetIntrustionDetectionReportCsvDownload`. Note the operationIds are spelled
   `Intrustion`, not `Intrusion` — use them verbatim.
4. **Image scanning** (per app): `ListCodeScanResultsForApp`
   (`GET /apps/{app_id}/code_scan_results`) → `GetCodeScanResult`. Results carry
   `dockerfile_present`, `procfile_present`, `aptible_yml_present`.

## Rules

- Intrusion-detection reports are only exposed when the stack has
  `expose_intrusion_detection_reports` set — check the `stack` payload before
  concluding a stack has none.
- Downloads are signed URLs; do not persist them.
- Reports are per-window (`starts_at` / `ends_at`); page with `page` / `per_page`
  and stitch by window rather than assuming one report covers the audit period.
