# AssistantMail Skill

This repository packages the AssistantMail AI agent skill. It is intended for agent frameworks that can load a skill definition and connect to an MCP server so the agent can discover mailboxes, read messages, and send email through the AssistantMail API.

The skill itself lives in [assistant-mail/SKILL.md](assistant-mail/SKILL.md).

## What This Skill Does

The skill gives an agent a stable way to:

- discover AssistantMail API access requirements
- list and inspect mailboxes
- list and fetch messages
- send email from a mailbox
- manage allowed recipients
- inspect mailbox usage

It is designed around the `assistantmail-mcp` command and the AssistantMail API key flow.

## Repository Layout

- [README.md](README.md): repository-level documentation
- [assistant-mail/SKILL.md](assistant-mail/SKILL.md): the skill definition consumed by agent tooling

## Access Model

The most important implementation detail is that mailbox access is based on `mailboxId`, not the mailbox email address.

- Knowing an address like `agent-123@assistant-mail.ai` does not grant access.
- Mailbox routes require the mailbox UUID in the path.
- Authorization is based on the owning account and a valid AssistantMail credential.

For agents, that means:

1. A human owner signs in.
2. A human owner creates an AssistantMail API key.
3. The API key is provided to the agent runtime securely.
4. The agent calls `GET /v1/mailboxes` to discover the mailbox it should use.
5. The agent stores and reuses the returned `mailboxId` for mailbox operations.

## Requirements

You need:

- access to an AssistantMail account
- an AssistantMail API key in the `amk_...` format
- an MCP-compatible client or agent runtime
- the `assistantmail-mcp` command available in that runtime

Optional environment variables:

- `ASSISTANT_MAIL_API_BASE_URL`: overrides the API base URL and defaults to `https://api.assistant-mail.ai`
- `ASSISTANT_MAIL_API_KEY`: default API key used by direct MCP tool calls when `apiKey` is omitted

## MCP Integration

Configure your client to launch the MCP server command:

```text
assistantmail-mcp
```

The exact registration format depends on the client:

- OpenClaw: register `assistantmail-mcp` in the MCP or skill registry
- Claude-compatible MCP clients: add an MCP server entry that launches `assistantmail-mcp`
- OpenAI-compatible MCP clients: configure an MCP connector that starts `assistantmail-mcp`

## Available Tools

The skill exposes the following operational tools:

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

## Example MCP Calls

List mailboxes:

```json
{
	"tool": "assistantmail_list_mailboxes",
	"input": {
		"apiKey": "amk_..."
	}
}
```

Get a mailbox:

```json
{
	"tool": "assistantmail_get_mailbox",
	"input": {
		"mailboxId": "<mailbox-uuid>",
		"apiKey": "amk_..."
	}
}
```

List messages:

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

Send email:

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

Get usage:

```json
{
	"tool": "assistantmail_get_usage",
	"input": {
		"mailboxId": "<mailbox-uuid>",
		"apiKey": "amk_..."
	}
}
```

If `ASSISTANT_MAIL_API_KEY` is set in the MCP server environment, `apiKey` can be omitted from the tool input.

## Notes

- API keys are created by a signed-in human owner; agents are expected to receive an existing `amk_...` key, not mint one themselves.
- API keys are only shown once when they are created.
- API key management endpoints are Cognito-authenticated, not API-key-authenticated.
- API key auth can be sent as either `x-api-key: amk_...` or `Authorization: Bearer amk_...`.
