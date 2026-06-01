---
name: assistant-mail
description: "A skill that allows AI agents to send emails using the AssistantMail API."
---
# AssistantMail AI Agent Skill

## Capability
This skill gives AI agents a consistent way to discover AssistantMail endpoint details and directly fetch mailbox/message data through MCP.

## Critical Access Model
- Knowing an email address like `agent-b1e4643c@assistant-mail.ai` does **not** grant mailbox access.
- Backend mail APIs are keyed by `mailboxId` (UUID path parameter), not by mailbox email address.
- Mailbox operations are authorized by account ownership (`mailbox.accountId === auth.accountId`).

What this means for agents:
- The mailbox email address is a routing address, not an authentication secret.
- Your client must call mailbox routes with the correct `mailboxId`.
- A valid API key or Cognito JWT is still required.

## Agent Bootstrap (Required)
1. Human owner signs in with Cognito.
2. Human owner creates an API key with `POST /v1/api-keys`.
3. Human owner shares the returned `amk_...` key securely with the agent runtime.
4. Agent lists mailboxes via `GET /v1/mailboxes` and stores the target `mailboxId`.
5. Agent uses that `mailboxId` for send/list/get/usage routes.

Notes:
- API keys are only shown once at creation.
- API key management endpoints are Cognito-only.
- API key auth can be supplied as `x-api-key: amk_...` or `Authorization: Bearer amk_...`.

## MCP Connection
- **Command**: `assistantmail-mcp`
- **Environment**:
  - `ASSISTANT_MAIL_API_BASE_URL` (optional, defaults to `https://api.assistant-mail.ai`)
  - `ASSISTANT_MAIL_API_KEY` (optional, `amk_...`; used by direct tools when `apiKey` input is omitted)
- **Available tools**:
  - `assistantmail_health`
  - `assistantmail_get_me`
  - `assistantmail_get_inbound_policy`
  - `assistantmail_update_inbound_policy`
  - `assistantmail_list_mailboxes`
  - `assistantmail_create_mailbox`
  - `assistantmail_get_mailbox`
  - `assistantmail_update_mailbox`
  - `assistantmail_delete_mailbox`
  - `assistantmail_list_messages`
  - `assistantmail_get_message`
  - `assistantmail_send_email`
  - `assistantmail_delete_messages`
  - `assistantmail_get_usage`
  - `assistantmail_list_recipients`
  - `assistantmail_add_recipient`
  - `assistantmail_remove_recipient`
  - `assistantmail_send_email_reference`
  - `assistantmail_list_messages_reference`
  - `assistantmail_get_message_reference`
  - `assistantmail_get_usage_reference`

These direct tools cover the API-key operational endpoints agents are expected to call.

## OpenClaw
Use this skill by registering the MCP command `assistantmail-mcp` in your OpenClaw skill/MCP registry.

## Claude
Use this skill by adding an MCP server entry that launches `assistantmail-mcp`.

## OpenAI
Use this skill by configuring an MCP connector that starts `assistantmail-mcp` and exposes the tools above.

## Direct MCP Calls (No Manual URL Building)
```json
{
  "tool": "assistantmail_list_mailboxes",
  "input": {
    "apiKey": "amk_..."
  }
}
```

```json
{
  "tool": "assistantmail_get_mailbox",
  "input": {
    "mailboxId": "<mailbox-uuid>",
    "apiKey": "amk_..."
  }
}
```

```json
{
  "tool": "assistantmail_list_messages",
  "input": {
    "mailboxId": "<mailbox-uuid>",
    "limit": 50,
    "since": "2026-01-01T00:00:00.000Z",
    "apiKey": "amk_..."
  }
}
```

```json
{
  "tool": "assistantmail_get_message",
  "input": {
    "mailboxId": "<mailbox-uuid>",
    "messageId": "<message-uuid>",
    "apiKey": "amk_..."
  }
}
```

```json
{
  "tool": "assistantmail_send_email",
  "input": {
    "mailboxId": "<mailbox-uuid>",
    "to": "recipient@example.com",
    "subject": "Hello",
    "text": "Hi there",
    "apiKey": "amk_..."
  }
}
```

```json
{
  "tool": "assistantmail_get_usage",
  "input": {
    "mailboxId": "<mailbox-uuid>",
    "apiKey": "amk_..."
  }
}
```

```json
{
  "tool": "assistantmail_list_recipients",
  "input": {
    "apiKey": "amk_..."
  }
}
```

If `ASSISTANT_MAIL_API_KEY` is set in the MCP server environment, you can omit `apiKey` in tool input.

## Minimal HTTP Example
```bash
# 1) Create API key (Cognito token required)
curl -X POST "$ASSISTANT_MAIL_API_BASE_URL/v1/api-keys" \
  -H "Authorization: Bearer <cognito-jwt>" \
  -H "Content-Type: application/json" \
  -d '{"name":"My Agent Key"}'

# 2) Discover mailboxId the agent will use
curl "$ASSISTANT_MAIL_API_BASE_URL/v1/mailboxes" \
  -H "x-api-key: <amk_key_from_step_1>"

# 3) Send from that mailboxId
curl -X POST "$ASSISTANT_MAIL_API_BASE_URL/v1/mailboxes/<mailboxId>/messages" \
  -H "x-api-key: <amk_key_from_step_1>" \
  -H "Content-Type: application/json" \
  -d '{"to":"recipient@example.com","subject":"Hello","text":"Hi there"}'
```
