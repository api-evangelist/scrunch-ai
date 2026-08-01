---
name: Run the Scrunch Optimize-and-Deploy pipeline
description: Audit pages, optimize content for AI search visibility, and deploy to AXP with one async pipeline, then poll for completion.
api: openapi/scrunch-ai-data-api-openapi.yml
operations: [createOptimizeAndDeployPipeline, getOptimizeAndDeployStatus, bulkRenderOptimizedPages]
auth: Bearer API key (configure scope)
---

# Run the Scrunch Optimize-and-Deploy pipeline

Drive Scrunch's asynchronous audit -> optimize -> deploy pipeline for a brand's pages.

## Prerequisites
- API key with the `configure` scope, as `SCRUNCH_API_TOKEN`.
- Base URL `https://api.scrunchai.com/v1`; `Authorization: Bearer $SCRUNCH_API_TOKEN`.

## Steps
1. **Submit the pipeline.** `createOptimizeAndDeployPipeline` (`POST /orchestration/optimize-and-deploy/{brand_id}`) with the list of page URLs. It returns immediately with a `token`.
2. **Track progress.** Either poll `getOptimizeAndDeployStatus` (`GET /orchestration/optimize-and-deploy/{brand_id}/{token}`) until complete, or subscribe to webhooks (`audit.completed`, `optimization.completed`, `deployment.completed`, `pipeline.completed` - see asyncapi/scrunch-ai-webhooks.yml).
3. **Bulk render (optional).** To (re)render optimized AXP pages for specific paths, call `bulkRenderOptimizedPages` (`POST /orchestration/render-bulk/{brand_id}/sites/{site_id}`); it queues render jobs and returns one job entry per path.

## Notes
- `bulkRenderOptimizedPages` may return `502` when jobs were queued but a downstream dispatch failed; the response lists pending jobs so you can retry the failures.
- `400` indicates a malformed payload or invalid input; `422` returns the `HTTPValidationError` envelope.
