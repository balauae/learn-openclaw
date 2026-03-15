# 20 — Plugins & Extensions

## What Are Plugins?

Plugins extend OpenClaw with new capabilities — new channels, memory backends, context engines, or tools. They're npm packages (or local directories) loaded at runtime.

---

## Plugin Types (by `kind`)

| Kind | What it does | Selected via |
|------|-------------|-------------|
| `"memory"` | Custom memory/search backend | `plugins.slots.memory` |
| `"context-engine"` | Custom context management | `plugins.slots.contextEngine` |
| channel plugin | Adds a new channel (e.g. Matrix, WeChat) | `channels.<id>` config |
| general | Adds tools, providers, skills | loaded by default if enabled |

---

## Installing Plugins

```bash
# From npm
openclaw plugins install @scope/package-name

# From a local directory
openclaw plugins install /path/to/my-plugin

# List installed
openclaw plugins list

# Remove
openclaw plugins remove <plugin-id>
```

---

## Plugin Manifest (`openclaw.plugin.json`)

Every plugin **must** have this file in its root:

```json
{
  "id": "my-plugin",
  "name": "My Plugin",
  "description": "Does something cool",
  "version": "1.0.0",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": { "type": "string" }
    }
  }
}
```

Optional fields:
- `kind` — plugin type (`"memory"`, `"context-engine"`)
- `channels` — channel ids this plugin registers (e.g. `["matrix"]`)
- `providers` — AI provider ids registered
- `skills` — skill directories to load (relative to plugin root)
- `uiHints` — labels/placeholders for config UI

**Important:** Schema is validated at config read/write time, not at runtime. Missing/broken manifest = plugin error + Doctor warning.

---

## Config

```json5
{
  plugins: {
    // Allow/deny specific plugins
    allow: ["my-plugin", "voice-call"],
    deny: [],

    // Slot assignments (exclusive plugins)
    slots: {
      memory: "my-memory-plugin",
      contextEngine: "my-context-plugin",
    },

    // Per-plugin config
    entries: {
      "my-plugin": {
        enabled: true,
        apiKey: "...",
      }
    }
  }
}
```

---

## Validation Rules

- Unknown `channels.*` keys → **error** (unless declared by a plugin manifest)
- `plugins.entries.<id>` with unknown id → **error**
- Plugin installed but disabled → **warning** in Doctor
- Plugin missing manifest → **error**, blocks config validation

---

## Skills vs Plugins

| | Skills | Plugins |
|--|--------|---------|
| Location | `~/.openclaw/skills/` | npm package or local dir |
| Purpose | AI task instructions | System capability extension |
| Format | `SKILL.md` + scripts | Node.js module + manifest |
| Distribution | Manual copy / ClawHub | npm install |
| Scope | AI behavior only | Can add channels, tools, providers |

---

## Community Plugins

Find plugins at [ClawHub](https://clawhub.com) or via npm.

Notable example:
- **WeChat** — Connect to WeChat via WeChatPadPro
  `openclaw plugins install @icesword760/openclaw-wechat`

Submit your own plugin: publish to npm + open PR to OpenClaw docs.

---

## Plugin Development

Structure:

```
my-plugin/
├── openclaw.plugin.json    ← manifest (required)
├── index.js                ← entry point
├── skills/                 ← optional bundled skills
└── README.md
```

The manifest is the contract. OpenClaw reads it without executing your code — so validation is safe and fast.
