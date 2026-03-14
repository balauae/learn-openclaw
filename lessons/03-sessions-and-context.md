# 03 — Sessions & Context

> How OpenClaw tracks conversations and manages context windows.

## What Is a Session?

A session is a named conversation thread. Each channel+sender combination gets its own session. Session keys look like:

```
agent:main:telegram:8523037700          # direct DM on Telegram
agent:main:discord:channel:123456789    # Discord channel
cron:job-abc123                         # isolated cron job session
```

Sessions persist their message history in `~/.openclaw/agents/<agentId>/sessions/`.

## Context Window

Every model has a finite context window (e.g. Claude Sonnet: 200k tokens). OpenClaw tracks token usage per session. As a session grows:

1. Old messages get summarised and compressed (**compaction**)
2. The summary is injected as context going forward
3. Recent messages remain verbatim

## Compaction

When a session approaches its context limit:

1. **Memory flush** fires first (silent agent turn: "write any important things to memory files now")
2. Old messages are summarised
3. Summary replaces the old messages in the context
4. Session continues with fresh headroom

Configurable via `agents.defaults.compaction`:
```json5
{
  reserveTokensFloor: 20000,   // keep this much headroom
  memoryFlush: {
    enabled: true,
    softThresholdTokens: 4000
  }
}
```

## Session Scope

- **Per-sender**: each person gets their own isolated session (default)
- **Global**: all messages share one session (`session.scope = "global"`)

## Listing Sessions

```bash
openclaw sessions           # list all sessions
openclaw sessions --json    # with keys and metadata
```

## Pruning

Old inactive sessions are pruned automatically. Configure via `agents.defaults.session.idleExpiry`.

## Isolated Sessions

Sub-agents, cron jobs, and spawned sessions each get their own isolated session that doesn't pollute the main history. They start with a clean context each run.
