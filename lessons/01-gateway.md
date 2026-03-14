# 01 — Gateway

> The heart of OpenClaw. Everything flows through it.

## What It Is

The Gateway is a long-running Node.js daemon (`openclaw gateway start`). It's the central hub that:
- Connects to all your messaging channels (Telegram, WhatsApp, Discord, etc.)
- Maintains sessions and context for each conversation
- Routes messages to the AI model and back
- Runs the scheduler (cron), heartbeat, and all background tasks

## How It Runs

```bash
openclaw gateway start    # start as background daemon
openclaw gateway stop     # stop it
openclaw gateway restart  # restart (picks up config changes)
openclaw gateway status   # check if it's running
```

The daemon persists across terminal sessions. It writes logs to `~/.openclaw/logs/`.

## WebSocket Protocol

Internally, OpenClaw uses a WebSocket-based protocol between components. The CLI (`openclaw`) connects to the running Gateway via a local WebSocket. This is how commands like `openclaw sessions` or `openclaw cron list` work — they're just sending messages to the Gateway over that socket.

## Config File

The Gateway reads `~/.openclaw/openclaw.json` (or `openclaw.jsonc` for comments). On startup it:
1. Validates the config schema
2. Starts all configured channel plugins
3. Arms the cron scheduler
4. Begins heartbeat cycles

## Key Directories

| Path | Purpose |
|---|---|
| `~/.openclaw/openclaw.json` | Main config |
| `~/.openclaw/workspace/` | Agent workspace (memory, skills, etc.) |
| `~/.openclaw/logs/` | Gateway logs |
| `~/.openclaw/cron/` | Cron job storage |
| `~/.openclaw/memory/` | Vector index (SQLite) |
| `~/.openclaw/agents/` | Per-agent state |
