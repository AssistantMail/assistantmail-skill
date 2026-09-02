# Assistant Mail — agent email with allowlist & consent

Managed agent email for personal and small-team OpenClaw and Hermes operators. Give each approved agent an address on assistant-mail.ai while you stay in control of **who they can email**, how long content is kept, and how much they can send — allowlist, consent, retention, and send caps.

This repository packages the Assistant Mail skill. Agent frameworks load the skill definition and connect to an MCP server so the agent can discover mailboxes, read messages, and send email through the Assistant Mail API.

The skill itself lives in [assistant-mail/SKILL.md](assistant-mail/SKILL.md).

**Listed on [ClawHub](https://clawhub.ai/assistantmail/skills/assistant-mail) — not an official OpenClaw endorsement.**

## Get started (Free)

Free: **1 mailbox**, **25 emails/day**.

1. [Create a Free account](https://app.assistant-mail.ai/?utm_source=github&utm_medium=readme&utm_campaign=clawhub_readme_amplify)
2. Create an API key in the dashboard (`amk_...`)
3. Install the skill from ClawHub:

```bash
openclaw skills install @assistantmail/assistant-mail
```

4. Configure MCP with `ASSISTANT_MAIL_API_KEY`

Skill page: [clawhub.ai/assistantmail/skills/assistant-mail](https://clawhub.ai/assistantmail/skills/assistant-mail)

Paid plans: **Upgrade in-app** from the dashboard (checkout is not via Payment Links). Outbound mail is gated by your recipient allowlist; on paid plans, consent invites must be confirmed before send.

Docs: [assistant-mail.ai/docs](https://assistant-mail.ai/docs)

## What This Skill Does

The skill gives an agent a stable way to:

- discover Assistant Mail API access requirements
- list and inspect mailboxes
- list and fetch messages
- send email from a mailbox
- manage allowed recipients (allowlist and consent)
- inspect mailbox usage

It is designed around the `assistantmail-mcp` command and the Assistant Mail API key flow.

## Repository Layout

- [README.md](README.md): repository-level documentation
- [assistant-mail/SKILL.md](assistant-mail/SKILL.md): the skill definition consumed by agent tooling and ClawHub

## Access Model

The most important implementation detail is that mailbox access is based on `mailboxId`, not the mailbox email address.

- Knowing an address like `agent-123@assistant-mail.ai` does not grant access.
- Mailbox routes require the mailbox UUID in the path.
- Authorization is based on the owning account and a valid Assistant Mail credential.

For agents, that means:

1. A human owner signs in.
2. A human owner creates an Assistant Mail API key.
3. The API key is provided to the agent runtime securely.
4. The agent calls `GET /v1/mailboxes` to discover the mailbox it should use.
5. The agent stores and reuses the returned `mailboxId` for mailbox operations.

## Requirements

You need:

- an Assistant Mail account ([Free signup](https://app.assistant-mail.ai/?utm_source=github&utm_medium=readme&utm_campaign=clawhub_readme_amplify): 1 mailbox, 25 emails/day)
- an Assistant Mail API key in the `amk_...` format
- an MCP-compatible client or agent runtime (OpenClaw, Hermes, Claude Desktop, Cursor, and similar)
- the `assistantmail-mcp` command available in that runtime

Optional environment variables:

- `ASSISTANT_MAIL_API_BASE_URL`: overrides the API base URL and defaults to `https://api.assistant-mail.ai`
- `ASSISTANT_MAIL_API_KEY`: default API key used by direct MCP tool calls when `apiKey` is omitted

## Install (OpenClaw / ClawHub)

Primary path for OpenClaw:

```bash
openclaw skills install @assistantmail/assistant-mail
```

Then register the MCP command `assistantmail-mcp` and set `ASSISTANT_MAIL_API_KEY` in the agent runtime. Hermes uses the same MCP server.

Paid upgrades are in-app only: [app.assistant-mail.ai](https://app.assistant-mail.ai/?utm_source=github&utm_medium=readme&utm_campaign=clawhub_readme_amplify).

## MCP Integration

Configure your client to launch the MCP server command:

```text
assistantmail-mcp
```

The exact registration format depends on the client:

- OpenClaw: install from ClawHub (command above), then register `assistantmail-mcp` in the MCP or skill registry
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
- `assistantmail_reply_message` – Reply to an email the agent has received. Handles threading (In-Reply-To, References) automatically.
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

Reply to a message:

```json
{
	"tool": "assistantmail_reply_message",
	"input": {
		"mailboxId": "<mailbox-uuid>",
		"messageId": "<message-uuid>",
		"text": "Thanks for your email!",
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
- Outbound send is gated by the recipient allowlist; paid plans require consent confirmation before the agent can send.
- Paid plans are upgraded in-app only.
