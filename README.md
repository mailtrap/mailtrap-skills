# Mailtrap user skills

Agent skills for [Mailtrap](https://mailtrap.io) end users and builders using LLM-assisted coding. They summarize product flows, API bases and paths, and links to official documentation. They are not a substitute for [docs.mailtrap.io](https://docs.mailtrap.io/) or the [developer API documentation](https://docs.mailtrap.io/developers/).

## Install

Copy or symlink the skill folders into the skills directory your agent reads (examples):

- **Cursor (project):** `.cursor/skills/` in your repo, e.g. `ln -s /path/to/mailtrap-skills/skills/* .cursor/skills/`
- **Cursor (user):** `~/.cursor/skills/` for global availability

Each skill is a directory containing `SKILL.md`. Restart or reload the agent if it does not pick up new skills immediately.

## Contents

| Skill                       | Purpose                                                     |
| --------------------------- | ----------------------------------------------------------- |
| `sending-emails`            | Live sending via Email API and SMTP (transactional vs bulk) |
| `testing-with-sandbox`      | Email Sandbox, Testing API, safe capture in dev/staging     |
| `using-email-templates`     | Account-scoped templates, Handlebars, API template send     |
| `setting-up-sending-domain` | Domain verification, DNS (add all records), compliance      |
| `managing-contacts`         | Contacts API, lists, segments, CRM-style sync               |

## References

- Product documentation: [docs.mailtrap.io](https://docs.mailtrap.io/)
- Developer and API documentation: [docs.mailtrap.io/developers](https://docs.mailtrap.io/developers/)
