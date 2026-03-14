# 13 — Thinking & Reasoning

## Thinking Levels

Control how hard the model thinks before answering:

| Level | Alias | Cost |
|---|---|---|
| `off` | — | Cheapest |
| `minimal` | "think" | Low |
| `low` | "think hard" | Medium |
| `medium` | "think harder" | Higher |
| `high` | "ultrathink" | Max |
| `adaptive` | — | Provider-managed (Claude 4.6 default) |

## How to Set It

**One message only:**
```
/think:high what's the best DB schema for this?
```

**Stick for the whole session** (send directive alone):
```
/think:medium
```
Clear with `/think:off`.

**Global default in config:**
```json5
agents: { defaults: { thinkingDefault: "low" } }
```

**Resolution order:** inline → session → config → adaptive fallback

## Reasoning Visibility

See the thinking process:
```
/reasoning on      → separate "Reasoning:" message after each reply
/reasoning stream  → live in Telegram draft bubble, disappears on send (Telegram only)
/reasoning off     → hidden (default)
```

## Verbose Mode

See tool calls as they happen:
```
/verbose on    → each tool call as its own message bubble
/verbose full  → tool calls + their outputs
/verbose off   → silent (default)
```

## Check Current State

```
/think       → current thinking level
/reasoning   → current reasoning visibility  
/verbose     → current verbose level
```

## In Cron Jobs

```bash
openclaw cron add ... --thinking high --model opus
```

Per-job thinking override — great for weekly deep-analysis tasks.

## Practical Guide

| Situation | Setting |
|---|---|
| Quick question | Default (adaptive) |
| Complex problem | `/think:high` |
| Debugging | `/think:medium` + `/verbose on` |
| See reasoning | `/reasoning on` |
| Live reasoning (Telegram) | `/reasoning stream` |
| Cost-sensitive bulk | `/think:off` |
