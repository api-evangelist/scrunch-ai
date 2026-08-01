---
name: Query Scrunch AI visibility metrics
description: Authenticate to the Scrunch Data API, resolve a brand, and pull AI visibility metrics (presence, sentiment, position, citations) across AI platforms.
api: openapi/scrunch-ai-data-api-openapi.yml
operations: [listBrands, query]
auth: Bearer API key (query scope)
---

# Query Scrunch AI visibility metrics

Use this skill to retrieve aggregated AI visibility metrics for a brand from Scrunch.

## Prerequisites
- A Scrunch API key with the `query` scope, exported as `SCRUNCH_API_TOKEN`.
- Base URL: `https://api.scrunchai.com/v1`.
- Send `Authorization: Bearer $SCRUNCH_API_TOKEN` on every request.

## Steps
1. **Resolve the brand.** Call `listBrands` (`GET /brands`, supports `limit`/`offset`) and pick the `brand_id` you want to report on. Brand-scoped keys only return their permitted brands.
2. **Query metrics.** Call `query` (`GET /{brand_id}/query`) with the dimensions and metrics you need (e.g. brand presence, sentiment score, position, competitor presence, source citations) across AI platforms and a date range. The endpoint dynamically aggregates observation data.
3. **Handle errors.** A `400` means an invalid date range or malformed parameters (dates must be `YYYY-MM-DD`); `422` returns an `HTTPValidationError` `detail[]` array; `404` means the brand or resource was not found.

## Notes
- Pagination is `limit`/`offset` (default limit 100).
- For raw answer text and citations rather than aggregates, use the `listResponses` skill.
