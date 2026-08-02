---
name: Export Unwrap feedback and taxonomy
description: Authenticate to the Unwrap GraphQL API, resolve your team/view, and page through feedback entries and the group taxonomy for analysis or sync.
api: https://data.api.production.unwrap.ai/
method: graphql
operations: [getOrganizations, entries, TeamGroupsTaxonomy]
source: https://docs.unwrap.ai/collections/6774268412-api
---

# Export Unwrap feedback and taxonomy

Use this skill to pull customer feedback out of Unwrap over its GraphQL API.

## Authentication
- Generate a personal API key from your Unwrap user profile (bottom-left → your name).
- Send it on every request as `Authorization: Bearer <UNWRAP_ACCESS_KEY>`.
- The endpoint is `https://data.api.production.unwrap.ai/`.
- Each token is bound to a single View (`team_id`); it expires if an org admin removes your membership.

## Steps
1. **Find your team id** — run the `getOrganizations` query to list your organization's teams and their `teamId`s.
2. **Page feedback entries** — run the `entries` query with `teamId` (Int, required). Optionally pass `filterInput` (`startDate`/`endDate`) to bound by date, or `modifiedSince` for incremental sync. Page with `take` and `skip`; `take` is capped at **100** per request, so loop `skip += 100` until fewer than 100 rows return.
3. **Pull the taxonomy** — run `TeamGroupsTaxonomy` with `teamId` and a required `filterInput`; it returns `amountOfGroups.amount` and nested `taxonomyTrees` (group id/title, uniqueEntries, up to ~5 nesting levels). Use `take`/`skip` to page.

## Rules
- Stay under **100 requests per token per minute** (see `rate-limits/unwrap-rate-limits.yml`).
- Handle GraphQL failures via the top-level `errors` array; see `errors/unwrap-problem-types.yml`.
- For incremental exports, persist the last `modifiedSince` timestamp and pass it on the next run rather than re-fetching everything.
