# Mailtrap user skills

Agent skills for [Mailtrap](https://mailtrap.io) end users and builders using LLM-assisted coding. They summarize product flows, API bases and paths, and links to official documentation. They are not a substitute for [docs.mailtrap.io](https://docs.mailtrap.io/) or the [developer API documentation](https://docs.mailtrap.io/developers/).

## Standard

These skills follow the [Agent Skills](https://agentskills.io) open standard (Anthropic, Dec 2025). Each skill is a directory containing a `SKILL.md` with YAML frontmatter (`name`, `description`) plus Markdown instructions. Compliant agents read only the frontmatter at startup and pull the body on demand (progressive disclosure), so the same folder works across multiple tools.

The repository is an [Agent Plugin](https://agent-plugins.org): root `plugin.json` plus `mcp.json`. A compatible client loads the skills under `skills/` and the Mailtrap MCP server from that one package. Native manifests for Claude Code, OpenAI Codex, and Cursor stay for clients that install those formats directly. Windsurf loads the same skills from its skills directory and does not require a plugin manifest.

## Install

### Plugin

Install this repository as one plugin in any [compatible client](https://agent-plugins.org/compatible-clients): Kiro, GitHub Copilot, Cursor, OpenClaw, VS Code, NanoClaw, Hermes Agent, Grok Bot, ChatGPT, and Codex.

Use the repository URL, or a local clone, in that client's plugin install flow (marketplace, Git URL, or local folder):

```text
https://github.com/mailtrap/mailtrap-skills
```

The client reads `plugin.json`, loads every skill under `skills/`, and starts the MCP server in `mcp.json` (`npx -y mcp-mailtrap`). Start a new session after install.

The plugin does not embed credentials. Set `MAILTRAP_API_TOKEN` in the client environment before using the MCP server. See [Tokens and account_id](#tokens-and-account_id).

### Manual install

If the client does not load Agent Plugins, copy or symlink the skill folders into the skills directory it reads. The examples below assume you cloned this repo to `/path/to/mailtrap-skills`. After install, restart or reload the agent if it does not pick up new skills.

#### Cursor

- Project: `.cursor/skills/<skill-name>/`
- User (all projects): `~/.cursor/skills/<skill-name>/`

```bash
ln -s /path/to/mailtrap-skills/skills/* ~/.cursor/skills/
```

#### Claude Code (Anthropic)

- Project: `.claude/skills/<skill-name>/`
- User: `~/.claude/skills/<skill-name>/`

```bash
ln -s /path/to/mailtrap-skills/skills/* ~/.claude/skills/
```

#### Codex CLI (OpenAI)

- Project: `.agents/skills/<skill-name>/`
- User: `~/.agents/skills/<skill-name>/`

```bash
ln -s /path/to/mailtrap-skills/skills/* ~/.agents/skills/
```

#### Gemini CLI (Google)

- Project: `.gemini/skills/<skill-name>/` (also accepts `.agents/skills/`)
- User: `~/.gemini/skills/<skill-name>/`

```bash
ln -s /path/to/mailtrap-skills/skills/* ~/.gemini/skills/
```

#### GitHub Copilot

- Project: `.github/skills/<skill-name>/` (also accepts `.claude/skills/` and `.agents/skills/`)
- User: `~/.copilot/skills/<skill-name>/` (also accepts `~/.agents/skills/`)

```bash
ln -s /path/to/mailtrap-skills/skills/* ~/.copilot/skills/
```

#### Windsurf (Cascade)

- Project: `.windsurf/skills/<skill-name>/`
- User: `~/.codeium/windsurf/skills/<skill-name>/`

```bash
ln -s /path/to/mailtrap-skills/skills/* ~/.codeium/windsurf/skills/
```

#### Other agents

Most other agents that support the Agent Skills standard also accept `~/.agents/skills/` (user scope) and `.agents/skills/` (project scope). If your tool uses a different path, point it at this repository's `skills/` folder.

## Tokens and account_id

Skill examples reference these environment variables instead of literal secrets:

| Variable                     | Used for                                                                              |
| ---------------------------- | ------------------------------------------------------------------------------------- |
| `MAILTRAP_API_TOKEN`         | General API: Email Send (transactional and bulk), Templates, Contacts, Sending Domains |
| `MAILTRAP_SANDBOX_API_TOKEN` | Sandbox / Email Testing (separate scope)                                              |
| `MAILTRAP_ACCOUNT_ID`        | Path parameter for account-scoped endpoints (resolve at runtime, do not hardcode)     |

Full guidance — token scope, where to store tokens, auth header forms, and how to resolve `MAILTRAP_ACCOUNT_ID` from `GET https://mailtrap.io/api/accounts` — lives in the `authorizing-api-requests` skill. Every other skill in this repo points at it instead of duplicating the rules.

## Contents

| Skill                          | Purpose                                                                                         |
| ------------------------------ | ----------------------------------------------------------------------------------------------- |
| `authorizing-api-requests`     | API tokens, env vars, auth headers, `account_id` resolution                                     |
| `sending-emails`               | Live sending via Email API and SMTP (transactional vs bulk)                                     |
| `testing-with-sandbox`         | Email Sandbox, Testing API, safe capture in dev/staging                                         |
| `using-email-templates`        | Account-scoped templates, Handlebars, API template send                                         |
| `converting-email-templates`   | Convert templates from SendGrid, Mailgun, Mandrill, Postmark, Brevo, SES to Mailtrap Handlebars |
| `setting-up-sending-domain`    | Domain verification, DNS (add all records), compliance                                          |
| `managing-contacts`            | Contacts API, lists, segments, CRM-style sync                                                   |
| `receiving-emails`             | Inbound Email: inboxes, webhooks, threads, replies, forwarding rules, agent inboxes             |

## References

- Product documentation: [docs.mailtrap.io](https://docs.mailtrap.io/)
- Developer and API documentation: [docs.mailtrap.io/developers](https://docs.mailtrap.io/developers/)
- Agent Skills standard: [agentskills.io](https://agentskills.io)
- Agent Plugins standard: [agent-plugins.org](https://agent-plugins.org)
- Mailtrap MCP server: [mcp-mailtrap](https://www.npmjs.com/package/mcp-mailtrap)
