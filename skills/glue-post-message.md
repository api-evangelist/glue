---
name: Post a message to a Glue thread
description: Authenticate to the Glue GraphQL API and send a message into an existing thread or create a new thread, idempotently.
api: https://api.gluegroups.com/public/graphql
protocol: graphql
operations: [sendMessage, groups, me]
source: https://docs.glue.ai/developers/graphql-api/messages_and_threading.md
generated: '2026-07-19'
method: generated
---

# Post a message to a Glue thread

Use this skill to send a message into a Glue workspace from an OAuth Application.

## 1. Authenticate

Exchange your OAuth Application client credentials for a workspace-scoped token at
`https://api.gluegroups.com/oauth/token` (`grant_type=client_credentials`, with
`client_id`, `client_secret`, and `subject` = the workspace ID). Requires the
`threads.messages:write` scope. There is no refresh token — request a new one when it expires.

Pass the token on every request: `Authorization: Bearer <ACCESS_TOKEN>`.

## 2. (Optional) Confirm context

- `me` — confirm the authenticated identity.
- `groups { nodes { id name } }` — list groups (IDs are prefixed `grp_`) to pick a recipient.

## 3. Send the message

Call the `sendMessage` mutation. To post into an existing thread, pass its `threadID`
(prefixed `thr_`) plus a `message` with `text` (markdown by default):

```graphql
mutation SendMessage($input: SendMessageInput!) {
  sendMessage(input: $input) {
    thread { id subject }
    message { id text }
  }
}
```

```json
{ "input": { "threadID": "thr_...", "message": { "text": "New bug reported!" } } }
```

To create a new thread instead, omit `threadID` and provide a `thread` object with
`recipients` (and a `subject` to force a thread in a group).

## 4. Make it idempotent

Retries are safe when you set the upsert keys:
- `thread.threadBy` — reuse instead of re-creating a thread.
- `message.uniqueBy` — reuse instead of re-posting a message.

Calling `sendMessage` again with the same key updates the existing object.

## Errors

GraphQL responses return an `errors[]` array alongside `data`; check it even on HTTP 200.
