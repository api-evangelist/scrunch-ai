---
name: Ingest AI agent traffic and referrals into Scrunch
description: Read aggregated AI bot/agent traffic for a site and push pre-aggregated AI-referral events from external analytics into Scrunch.
api: openapi/scrunch-ai-data-api-openapi.yml
operations: [getAgentTraffic, registerAiReferralsConnection, listAiReferralsConnections, pushAiReferralsEvents, disableAiReferralsConnection]
auth: Bearer API key (query + configure scopes)
---

# Ingest AI agent traffic and referrals into Scrunch

Read Scrunch's AI bot/crawler traffic and push external AI-referral data into the AI Traffic dashboard.

## Prerequisites
- API key (`query` to read, `configure` to register/push), as `SCRUNCH_API_TOKEN`.
- Base URL `https://api.scrunchai.com/v1`; `Authorization: Bearer $SCRUNCH_API_TOKEN`.

## Read agent traffic
1. `getAgentTraffic` (`GET /{brand_id}/sites/{site_id}/agent-traffic`) - aggregated bot/AI-agent traffic by date, path, agent source, and bot type.

## Push AI referrals
1. **Register a connection.** `registerAiReferralsConnection` (`POST /{brand_id}/ai-referrals/connections`) for a `(brand, website, provider)` tuple. Events can only be pushed once a connection exists. A `409` means a connection or conflicting integration already exists.
2. **List connections.** `listAiReferralsConnections` (`GET /{brand_id}/ai-referrals/connections`) - push connections only (GA integrations live elsewhere).
3. **Push events.** `pushAiReferralsEvents` (`POST /{brand_id}/ai-referrals/connections/{connection_id}/events`) with a batch of pre-aggregated daily referral events. `413` = payload over 10,000 events or 5 MB; `422` = one or more events failed validation.
4. **Disable.** `disableAiReferralsConnection` (`DELETE /{brand_id}/ai-referrals/connections/{connection_id}`) soft-disables; retained events remain but further pushes return `410 Gone`.

## Notes
- Re-register a disabled connection before retrying pushes.
