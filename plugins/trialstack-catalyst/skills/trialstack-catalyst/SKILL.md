---
name: trialstack-catalyst
description: Use when a user asks Codex to select a TrialStack organization, inspect governed trial data, or prepare and explicitly approve a permitted TrialStack change through Catalyst MCP.
---

# TrialStack Catalyst

TrialStack Catalyst is the MCP layer between TrialStack and an AI client. It
provides organization-scoped, governed access to TrialStack records without
giving the client a general-purpose API credential.

## Choose the workflow

- When the user asks which account is connected or wants to change
  organizations, call `list_organizations`. Use the returned `organizationId`
  on every later organization-scoped tool call in that workflow.
- For exploration, use `search` for cross-entity discovery and `fetch` when the
  entity type and ID are known. Pass `trialId` to `search` when exploring
  trial-scoped collections such as Analysis Specifications or amendments.
  Search results use stable entity IDs; do not substitute snapshot/version IDs.
  If search reports failed collections, its results are incomplete: inspect
  those exact read operations before concluding that records are absent.
- For exact API access, call `search_operations`, inspect the contract with
  `describe_operation`, and use `call_read_operation` only when the operation is
  marked read-only.
- For changes, discover and describe the exact operation before calling
  `prepare_change`. Never skip directly to an execution tool.

## Read-only checklist

1. Restate which TrialStack organization data the user is trying to inspect.
2. Prefer the smallest relevant read tool or operation.
3. Report record identifiers and the operations used.
4. Treat an empty collection as a successful data result. Do not describe zero
   records as an authentication or connection failure.
5. Clearly separate TrialStack facts from assumptions or external context.

## Large run artifacts

Read run details with `includeArtifacts=false` when artifact content is not
needed. For action execution evidence, discover the `artifact-summaries`
operation, page with `nextCursor` as `after`, and select an immutable artifact ID.
Current versions are the default; use `currentOnly=false` to inspect history.
Read the selected artifact's `content` endpoint and follow `nextOffset` until
null. Concatenate every chunk before parsing JSON. Offsets count Unicode
characters, so use the returned offset instead of a JavaScript string length.

Results larger than 16 KiB are available once in `structuredContent.result`.
Results above 1 MiB return a bounded response-delivery error with no payload.
The operation has already executed: never repeat a change because its response
was too large. Inspect the persisted record or run using bounded reads.

## Governed-change checklist

Changes are always two-step. Call `prepare_change` with the complete operation
input, present its preview and effect to the user, and call the matching
effect-specific execution tool only after explicit approval in a follow-up user
message. Pass the same organization ID, operation ID, and input with the returned
intent ID. Intents expire after ten minutes and cannot be replayed.

Use the execution tool that matches the prepared effect:

| Prepared effect | Execution tool            |
| --------------- | ------------------------- |
| `create`        | `execute_create_change`   |
| `update`        | `execute_update_change`   |
| `archive`       | `execute_archive_change`  |
| `restore`       | `execute_restore_change`  |
| `approve`       | `execute_approval_change` |
| `action`        | `execute_action_change`   |
| `async`         | `execute_async_change`    |

Never prepare and execute a change in response to the same user message. If the
preview differs from the user's request, prepare a new intent rather than
editing or reusing the old one.

## Critical rules

Never ask for or expose OAuth tokens. OAuth authenticates the user, while each
tool call selects an organization by its server-returned `organizationId`.
Catalyst verifies membership, plan access, and the user's current TrialStack
capabilities for that organization on every call. The OAuth organization is a
default, not a permanent connection boundary.
Treat unavailable operations as a policy boundary; do not look for an ungoverned
HTTP workaround.

Do not infer that a connection failed just because a collection has no records.
Authentication failures return an OAuth error; authorization failures return a
permission error; a successful empty response means the selected organization
currently has no matching data.

## Verify the result

- Confirm the request stayed within the selected organization.
- For reads, state whether the result was populated or successfully empty.
- For prepared changes, show the operation, effect, complete input, preview, and
  intent expiry before asking for approval.
- After execution, report the returned TrialStack record or run identifier and
  do not replay the intent.
