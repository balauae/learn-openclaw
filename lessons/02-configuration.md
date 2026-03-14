# 02 — Configuration

> openclaw.json — the single source of truth for everything.

## File Format

`~/.openclaw/openclaw.json` (or `.jsonc` for comments). Uses JSON5-like syntax in some places.

## Key Top-Level Sections

```json5
{
  agents: { ... },      // agent behaviour, heartbeat, memory, tools
  channels: { ... },    // telegram, discord, whatsapp, etc.
  models: { ... },      // provider API keys, model defaults
  cron: { ... },        // scheduler settings
  memory: { ... },      // memory backend config
  plugins: { ... },     // enable/disable plugin slots
  tools: { ... },       // tool policies
  security: { ... },    // allowlists, sandbox
}
```

## config.patch vs config.apply

Two ways to update config via the Gateway tool:

| | `config.patch` | `config.apply` |
|---|---|---|
| What it does | Partial update, merges with existing | Replaces entire config |
| Safety | ✅ Safer — only touches what you specify | ⚠️ Overwrites everything |
| Use when | Changing one or two fields | Replacing whole config |

Both trigger a restart after writing.

## Schema Inspection

Before editing, look up the schema for a specific path:

```
gateway → config.schema.lookup → path: "agents.defaults.heartbeat"
```

This tells you valid fields and types without guessing.

## Viewing Current Config

```
gateway → config.get
```

Returns the live running config.

## Practical Example — Patching Heartbeat Interval

```json5
// config.patch
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "every": "1h"
      }
    }
  }
}
```

Only the heartbeat interval changes; everything else untouched.

## Per-Agent vs Defaults

Most settings follow this pattern:
- `agents.defaults.*` — applies to all agents
- `agents.list[].{setting}` — overrides for a specific agent

If any `agents.list[]` entry has a setting (e.g. `heartbeat`), it takes over for that agent only.
