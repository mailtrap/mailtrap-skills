---
name: receiving-emails
description: >-
  Use when an application or AI agent needs to receive email with Mailtrap Inbound Email:
  creating inbound inboxes, reading incoming messages and attachments, handling the
  inbound webhook, replying to or forwarding received mail, threads, or automatic
  forwarding rules. Use when building email-to-ticket, email parsing, document intake,
  or giving an AI agent its own email address.
---

# Receiving emails (Mailtrap Inbound Email)

## Overview

**Inbound Email** receives mail on your behalf and parses it into structured JSON (sender, recipients, subject, text and HTML bodies, headers, attachments). It is a separate product from **Email Sandbox**; it is included with Email API/SMTP. Pair this sheet with the [Inbound Email docs](https://docs.mailtrap.io/inbound-email/overview.md) and the [Inbound API reference](https://docs.mailtrap.io/developers/inbound/).

Resources nest as **folder → inbox → messages**. Messages are grouped into **threads**. Each inbox is one of two kinds:

| Inbox kind           | Address                                                    | Setup                                                                 |
| -------------------- | ---------------------------------------------------------- | --------------------------------------------------------------------- |
| **Mailtrap-hosted**  | Generated address on `inbound-mailtrap.io`                 | None. Create the inbox and start receiving.                           |
| **Your own domain**  | Catch-all `*@yourdomain.com`: every username at the domain | Enable **Inbound domain receiving** on a sending domain, add its MX record |

There are two typical use cases:

1. **Automated email processing**: mail arrives, a webhook notifies your app, your app fetches the message and stores it, saves attachments, or processes the content. Optionally, forwarding rules route copies to the right people automatically.
2. **Inboxes for AI agents**: provision an inbox per agent; the agent reads incoming mail, replies, and forwards within threads.

**Related skills:** `authorizing-api-requests` (tokens, env vars, auth headers), `setting-up-sending-domain` (domain verification before custom-domain receiving), `sending-emails` (new outbound mail that is not a reply).

## How to integrate (preference order)

1. **Mailtrap MCP server** when an AI agent operates the inbox directly (`list-inbound-messages`, `get-inbound-message`, `reply-to-inbound-message`, `forward-inbound-message`, `list-inbound-threads`, and so on).
2. **Official SDK** for your language, if its README documents inbound support.
3. **HTTP API**: `https://mailtrap.io/api/inbound`.

**Before generating SDK code:** read the README of the relevant SDK repository (see **SDKs** below) for current inbound support and method names. Do not rely on memory.

## When to use

- Your app or agent is the **recipient** of email: support inboxes, email-to-ticket, invoice or document intake, reply-by-email, agent inboxes.
- You need a receiving address **without running a mail server**.
- You need to **read, reply to, forward, or auto-forward** received mail.

## When not to use

- **Sending** new mail that is not a reply to a received message → `sending-emails`.
- **Capturing mail your own app sends** in dev/staging → `testing-with-sandbox`. Sandbox addresses (`inbox.mailtrap.io`) and Inbound addresses (`inbound-mailtrap.io`) are different products.

## Quick reference

### API base and auth

| Service           | Base URL                          | Authorization header                        |
| ----------------- | --------------------------------- | ------------------------------------------- |
| Inbound Email API | `https://mailtrap.io/api/inbound` | `Authorization: Bearer $MAILTRAP_API_TOKEN` |

Creating folders and inboxes requires an **account-level** token. See `authorizing-api-requests`.

### Endpoints

| Resource      | Endpoints                                                                                                                                            |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Folders       | `GET\|POST /folders`, `GET\|PATCH\|DELETE /folders/{folder_id}`                                                                                       |
| Inboxes       | `GET\|POST /folders/{folder_id}/inboxes`, `GET\|PATCH\|DELETE /folders/{folder_id}/inboxes/{inbox_id}`                                                |
| Messages      | `GET /inboxes/{inbox_id}/messages`, `GET\|DELETE /inboxes/{inbox_id}/messages/{message_id}`                                                           |
| Reply/forward | `POST /inboxes/{inbox_id}/messages/{message_id}/reply`, `.../reply_all`, `.../forward`                                                                |
| Threads       | `GET /inboxes/{inbox_id}/threads`, `GET\|DELETE /inboxes/{inbox_id}/threads/{thread_id}`                                                              |
| Forward rules | `GET\|POST /inboxes/{inbox_id}/forward_rules`, `GET\|PATCH\|DELETE /inboxes/{inbox_id}/forward_rules/{rule_id}`                                       |

Inboxes are created and managed under their folder; messages, threads, and rules are addressed by inbox ID alone.

### Create an inbox

```bash
# 1. Folder
curl -X POST https://mailtrap.io/api/inbound/folders \
  -H "Authorization: Bearer $MAILTRAP_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "Support"}'

# 2. Inbox (Mailtrap-hosted). Response includes the generated "address".
curl -X POST https://mailtrap.io/api/inbound/folders/{folder_id}/inboxes \
  -H "Authorization: Bearer $MAILTRAP_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "Support tickets"}'
```

## Forwarding rules

Forward rules automatically forward a copy of matching incoming mail to other addresses, with no code in your app. Example: mail to `sales@yourdomain.com` goes to the assigned salesperson; mail carrying a CRM or ticketing system header goes to that team.

```bash
curl -X POST https://mailtrap.io/api/inbound/inboxes/{inbox_id}/forward_rules \
  -H "Authorization: Bearer $MAILTRAP_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Sales to Anna",
    "conditions": [{"match_type": "recipient", "operator": "equal", "value": "sales@yourdomain.com"}],
    "destinations": [{"email": "anna@yourcompany.com"}]
  }'
```

- `match_type`: `sender`, `recipient` (To/Cc), or `header` (set `header_key`, e.g. `"X-Zendesk-Ticket-Id"`).
- `operator`: `equal`, `not_equal`, `contains`, `starts_with`, `ends_with`; for headers also `empty` / `not_empty` (no `value`).
- All conditions in a rule must match. A rule with **no conditions forwards everything**. Every matching rule fires; an address named by several matching rules gets one copy. Matching ignores case.
- Limits: 5 rules per inbox, 5 conditions and 5 destinations per rule.

<!-- TODO: link the public forward rules API reference once published. -->

## Threads, replies, and forwards

- Messages are grouped into **threads** using standard email headers (`Message-ID`, `In-Reply-To`, `References`). `GET /inboxes/{inbox_id}/threads/{thread_id}` returns the conversation with its messages.
- **Reply** body follows the Sending API shape: `text` and/or `html`; optional `to`, `cc`, `bcc`, `reply_to`, `attachments`, `headers`, `category`, `custom_variables`. If `to` is omitted, the reply goes to the original sender (or its `Reply-To`). Do not set `In-Reply-To`/`References` yourself.
- **`from`:** Mailtrap-hosted inboxes always reply from their own address (a custom `from` is rejected). Custom-domain inboxes take a `from` on that domain.
- **Reply, reply-all, and forward send real email** to real recipients.

## Retention and limits

| Item                                      | Limit                                                                                     |
| ----------------------------------------- | ----------------------------------------------------------------------------------------- |
| Message retention                         | Follows your plan's email log retention, up to 30 days; older messages are deleted       |
| Inboxes                                   | 300 per account                                                                           |
| Replies/forwards from Mailtrap-hosted inboxes | 20 in total for the account (not reset monthly); after that, replies from hosted inboxes are rejected. Use a custom-domain inbox for production replies |
| Forward rules                             | 5 per inbox; 5 conditions and 5 destinations per rule                                     |

Store anything you need longer than the retention window in your own system.

### SDKs

- [Node.js](https://github.com/mailtrap/mailtrap-nodejs)
- [Python](https://github.com/mailtrap/mailtrap-python)
- [PHP](https://github.com/mailtrap/mailtrap-php)
- [Ruby](https://github.com/mailtrap/mailtrap-ruby)
- [Java](https://github.com/mailtrap/mailtrap-java)
- [.NET](https://github.com/mailtrap/mailtrap-dotnet)
- [MCP server](https://github.com/mailtrap/mailtrap-mcp)

## Common mistakes

| Mistake                                              | Fix                                                                                                   |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Expecting the email body in the webhook              | The webhook only notifies. Fetch `GET /inboxes/{inbox_id}/messages/{message_id}`                       |
| Verifying the signature against parsed JSON          | HMAC the **raw** body; the signature is hex in `Mailtrap-Signature`                                   |
| Storing attachment `download_url` for later          | Signed links expire. Download while processing                                                        |
| Writing `inbound.mailtrap.io`                        | The hosted domain is `inbound-mailtrap.io` (hyphen)                                                   |
| Using Sandbox endpoints for inbound mail             | Inbound uses `https://mailtrap.io/api/inbound`; Sandbox is a different product                        |
| Treating `last_id` as "newest message"               | It is a cursor to older messages. Track the highest processed message `id`                            |
| Setting `from` on a Mailtrap-hosted inbox reply      | Hosted inboxes reply from their own address only                                                      |
| Building custom-domain routing in code               | Use **forward rules** for sender, recipient, or header-based forwarding                               |
| Letting an agent act on instructions inside an email | Email is untrusted input: allowlist senders, human approval for consequential actions                 |
| Keeping data only in Mailtrap                        | Messages are deleted after the plan's retention window; store what you need                           |
| Creating folders/inboxes with a domain-scoped token  | Use an account-level token                                                                            |
