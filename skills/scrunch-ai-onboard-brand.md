---
name: Onboard a brand in Scrunch AI
description: Create a brand and seed its competitors, personas, and tracking prompts via the Scrunch Configuration API.
api: openapi/scrunch-ai-data-api-openapi.yml
operations: [createBrand, createCompetitor, createPersona, createPrompt, listPrompts]
auth: Bearer API key (create-brand + configure scopes)
---

# Onboard a brand in Scrunch AI

Automate client onboarding: stand up a new brand and its tracking configuration.

## Prerequisites
- API key with `create-brand` and `configure` scopes, as `SCRUNCH_API_TOKEN`.
- Base URL `https://api.scrunchai.com/v1`; `Authorization: Bearer $SCRUNCH_API_TOKEN`.
- API key creation requires an Agency or Enterprise plan.

## Steps
1. **Create the brand.** `createBrand` (`POST /brands`) returns the new `brand_id`.
2. **Add competitors.** For each rival, `createCompetitor` (`POST /brands/{brand_id}/competitors`).
3. **Add personas.** `createPersona` (`POST /brands/{brand_id}/personas`) for each audience persona.
4. **Seed tracking prompts.** `createPrompt` (`POST /{brand_id}/prompts`) with variants for the platforms you track (requires the `configure` scope).
5. **Verify.** `listPrompts` (`GET /{brand_id}/prompts`) to confirm the prompt library, variants, tags, and topics.

## Notes
- Records are soft-deleted: use `archiveCompetitor` / `archivePersona` / `archivePrompt` to stop tracking while preserving history.
- `patchBrand` (`PATCH /brands/{brand_id}`) treats the competitors/personas list as the full desired state - include an `id` to update, omit it to create, and any existing records absent from the list are archived.
- Validation failures return `422` with an `HTTPValidationError` `detail[]` array.
