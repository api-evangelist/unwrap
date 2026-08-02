---
name: Batch-import feedback and build groups in Unwrap
description: Push customer feedback into an Unwrap view via idempotent batch import, then classify it by building async groups from natural-language descriptions.
api: https://data.api.production.unwrap.ai/
method: graphql
operations: [BatchImport, buildGroupAsync]
source: https://docs.unwrap.ai/collections/6774268412-api
---

# Batch-import feedback and build groups in Unwrap

Use this skill to load feedback into Unwrap and organize it.

## Authentication
- `Authorization: Bearer <UNWRAP_ACCESS_KEY>` against `https://data.api.production.unwrap.ai/`.
- Token is scoped to one View (`teamId`).

## Import feedback (idempotent upsert)
- Call the `BatchImport` mutation: `BatchImport($entries: [BatchEntryInput!]!, $teamId: Int!)`; it returns `entriesScheduledForImport`.
- Each `BatchEntryInput` carries `providerUniqueId`, `title`, `fullText`, `submitterType` (customer/agent), `dataSource`, `dataSourcePermalink`, `feedbackDate` (ISO 8601), `feedbackSubmitterAlias`, optional `conversationParts`, and optional `customFields`.
- **Idempotency:** `providerUniqueId` is the natural key — re-submitting the same id updates rather than duplicates the entry (see `conventions/unwrap-conventions.yml`).
- **Limits:** ≤ 500 entries per request, ≤ 10 MB payload, 30-second timeout, 100 req/min/token. Expect up to a 24-hour delay between scheduling and the entry appearing fully imported.

## Build a classification group
- Call `buildGroupAsync` with `teamId`, `title`, `description` (natural-language theme driving the classifier), optional `parentGroupId`, `scopeToParentEntries`, and an optional `filterNode` metadata AST **sent as stringified JSON** (operators: `==`, `!=`, `in`, `!in`, `exists`, `not_exists`, `>=`, `<=`).
- It returns `groupId`, `jobId`, and `conversationId`; classification typically completes in 5–60 minutes depending on dataset size — poll rather than block.

## Rules
- Chunk imports to ≤ 500 entries; retry a whole failed batch (safe because import is keyed on `providerUniqueId`).
- Surface GraphQL `errors[]` to the caller; map size/format/timeout failures per `errors/unwrap-problem-types.yml`.
