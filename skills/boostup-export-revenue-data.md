---
name: Export Boostup revenue data
description: >-
  Pull Opportunities, Accounts, and Forecast Submissions out of the Boostup /
  Terret revenue intelligence platform via the Export API, with API-key auth,
  date filtering, and limit/skip pagination.
api: openapi/boostup-openapi-original.json
operations:
- export_opportunity_export_opportunity_post
- export_account_export_account_post
- export_forecast_submission_export_forecast_submission_post
---

# Export Boostup revenue data

Use the Boostup Export API (`https://app.boostup.ai/export`) to extract your
revenue data as JSON for in-house analysis. All three endpoints are read-only
exports; POST carries the filter body.

## Authenticate

- Create an API key in the API Key Service portal at `/settings/external-api-token`.
  If you cannot access it, ask your CSM to grant API access.
- Send the key verbatim in the `Authorization` header on every request. A key
  minted by a user stops working if that user is deactivated.
- A `401` means a missing/invalid key; a `403` means the key lacks access to
  that specific export service.

## Export Opportunities — `export_opportunity_export_opportunity_post`

- `POST /export/opportunity` with a JSON body.
- Required: `limit` (max page size 1000) and `skip` (offset).
- Optional date filters: `created_date`, `closed_date`, `updated_at` — each a
  `{ start_date, end_date? }` object (`start_date` required).
- Paginate: keep incrementing `skip` by `limit` until fewer than `limit`
  records come back.

## Export Accounts — `export_account_export_account_post`

- `POST /export/account`. Required `limit` + `skip`.
- Optional filters: `created_date`, `updated_at`, `email_domain`, `account_id`.

## Export Forecast Submissions — `export_forecast_submission_export_forecast_submission_post`

- `POST /export/forecast_submission`. Required `limit` + `skip`.
- Optional filters: `submission_type`, `business_type`, `forecast_name`, `year`,
  `quarter`, `month`, `week`, `period_type`, `user`, `created_date`.
- Each submission carries `included_deals` / `excluded_deals`
  (`opportunity_id` + `opportunity_name`) — join these back to exported
  Opportunities to reconcile the forecast number.

## Handle errors and limits

- `422` — validation failed; read `detail[].loc` / `detail[].msg`. `limit` and
  `skip` are required; date filters require `start_date`.
- `429` — rate limited; back off and paginate in smaller windows rather than
  requesting large date ranges at once.
- Error bodies are a plain JSON object with a `detail` field (not RFC 9457).
