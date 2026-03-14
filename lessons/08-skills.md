# 08 — Skills

## What Are Skills?

Skills are drop-in capability packages — a directory with a `SKILL.md` that tells the agent how to use some tool or API. Think of them as the agent's playbook for specific tasks.

## Discovery

The system prompt lists available skills with their descriptions. When a task matches, the agent reads the skill's `SKILL.md` and follows it.

## SKILL.md Structure

```markdown
# Skill Name

## When to use
...

## How to use
...

## Steps
1. ...
2. ...
```

The agent reads this file on demand — skills aren't always loaded, only when needed.

## Built-in Skills (your setup)

| Skill | Trigger |
|---|---|
| `coding-agent` | Build features, review PRs, refactor code |
| `weather` | Weather/forecast questions |
| `comfyui` | Generate images |
| `healthcheck` | Security audits, system hardening |
| `skill-creator` | Create or improve skills |

## Skill Location

Skills live in:
- `~/.nvm/.../openclaw/skills/` — built-in
- `~/.openclaw/skills/` — custom/user skills

## Creating a Skill

Use the `skill-creator` skill itself:
> "Create a skill for checking my Docker containers"

It'll scaffold the directory, write a `SKILL.md`, and optionally add helper scripts.

## ClawHub

Community skills at [clawhub.com](https://clawhub.com) — install like packages.
