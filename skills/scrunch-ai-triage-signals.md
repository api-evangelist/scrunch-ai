---
name: Triage Scrunch AI visibility signals
description: Pull Scrunch's nightly detection sweep, read what changed in a brand's AI visibility, and record the team's reaction on each signal.
api: openapi/scrunch-ai-signals-api-openapi.yml
operations: [listSignalAnchors, listSignals, getSignal, setSignalReaction, clearSignalReaction, listSignalReactions]
generated: '2026-08-13'
method: generated
source: https://developers.scrunch.com/api-reference/signals/overview
---

# Triage Scrunch AI visibility signals

Scrunch runs a nightly detection sweep across a brand's AI visibility metrics and emits a
**signal** for each statistically-tested movement — a level change or a trend — on a slice
of the data. Use this skill to pull what changed, decide what matters, and close the loop
with a reaction the rest of the team can see.

Base URL: `https://api.scrunchai.com/v1`

## Authentication

Every call needs a Bearer token in the `Authorization` header:

```
Authorization: Bearer $SCRUNCH_API_TOKEN
```

Read operations need the **`query`** scope. **Reaction writes are different**: they require a
**user token (JWT)**, because a reaction is attributed to a person. An organization API key
has no user identity and receives **HTTP 403** on `setSignalReaction` and
`clearSignalReaction`. Do not retry a 403 here — it is a credential-class mismatch, not a
transient failure. API key provisioning requires the Agency or Enterprise plan; without it,
key creation returns **402 Payment Required**.

## Steps

1. **Find the days that produced signals.** Call `listSignalAnchors` for the brand. It
   returns the distinct detection dates, which is what you drive a date picker or a polling
   loop from — do not guess dates or sweep a blind range.

2. **List what changed.** Call `listSignals` with the filters that match the question:
   - `scope` — `account`, `account_platform`, `topic`, or `topic_platform`
   - `metric` — `presence_rate`, `position_top_rate`, or `cited_domain_rate`
   - `alert_type` — `level_change` (step shift) or `trend` (sustained drift)
   - `direction` — `up`, `down`, or `none`
   - `subject_kind` — `brand` or `competitor`
   - `tier` — omit it to get the user-facing tiers (`high`, `confident`, `worth_a_look`,
     `provisional`); pass an explicit tier to reach the noise-floor tiers
     (`low_confidence`, `underpowered`, `untested`)
   - `platform` — e.g. `OpenAI`. Multi-platform signals carry a `(multi)` sentinel; read
     the real list from `slice.platforms` on the response, not from the filter.
   - `anchor_from` / `anchor_to` — inclusive `detected_for_date` bounds, `YYYY-MM-DD`
   - `sort` — `score_desc` (default, engine priority), `delta_desc`, or `detected_desc`

   Pagination is `limit`/`offset`, default `50`, **max `200`**. Results are deduplicated to
   the latest detection per identity, so a multi-day range returns one row per underlying
   issue rather than one row per nightly re-detection.

3. **Read the narrative before acting.** Each signal carries `narrative.what_happened`,
   `narrative.why_it_matters`, and `narrative.what_to_do`, plus `current_value`,
   `baseline_value`, `delta_absolute`, `baseline_definition` and the comparison window
   (`window_current_start` / `window_current_end`). Quote these rather than recomputing a
   delta — the baseline definition is the provider's, not yours.

4. **Pull the full record when you need detail.** Call `getSignal` with the `signal_id` for
   one signal's complete narrative and its `url_movers` (per-URL citation movers, emitted
   for `cited_domain_rate` signals).

5. **Track identity across nights with the fingerprint.** Every signal has a stable
   `fingerprint` for the underlying issue that survives nightly re-detection. Key your own
   state on the fingerprint, **not** on `id` — that is what keeps a follow-up from being
   filed twice for the same issue.

6. **Record the team's reaction.** Call `setSignalReaction` (PUT) with one of `useful`,
   `not_useful`, `dismissed`, or `actioned`, plus an optional free-text reason. It replaces
   any existing reaction from the same caller. Use `clearSignalReaction` (DELETE) to remove
   it.

7. **Report on the loop.** Call `listSignalReactions` for brand-wide reporting; it filters
   by `fingerprint` (every reaction on one signal identity) or by `reaction` value.

## Rules and gotchas

- **No idempotency contract.** Scrunch documents no `Idempotency-Key` header or parameter
  anywhere in the API. `setSignalReaction` is a PUT and is naturally idempotent by replacing
  the caller's reaction; nothing else here is. Never blind-retry a write.
- **No published rate limits.** Scrunch documents no rate limits and returns no
  `RateLimit-*` / `Retry-After` headers, so there is no runtime signal to back off on.
  Be conservative: page with `limit` at or below the 200 cap and avoid tight polling loops.
- **Errors are not RFC 9457.** Validation failures return a FastAPI `HTTPValidationError`
  envelope as `application/json`, not `application/problem+json`. See
  `errors/scrunch-ai-problem-types.yml`.
- **Signals are REST-only.** None of the 33 published Scrunch MCP tools expose the signals
  surface, so an MCP client cannot reach it — see `mcp/scrunch-ai-tool-crosswalk.yml`.
- **Detection thresholds are the point.** For ad-hoc "how is metric X trending" questions
  use the Query API (`query`) instead; Signals only surfaces movements that passed
  detection.

## Related

- `conventions/scrunch-ai-conventions.yml` — pagination, auth style, token identity
- `examples/scrunch-ai-examples.yml` — verbatim published request/response samples
- `authentication/scrunch-ai-authentication.yml` — key scopes and plan gating
