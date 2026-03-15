# 19 — Channels: Discord, WhatsApp & Signal

## Overview

OpenClaw supports many messaging channels. They all share the same pattern:
- Configure in `openclaw.json` under `channels.<name>`
- Control access with `dmPolicy`, `allowFrom`, `groupPolicy`
- Connect/auth with `openclaw channels login --channel <name>`

---

## Discord

**Method:** Official Discord Bot API (Gateway WebSocket)
**Status:** Production-ready. Supports DMs + guild channels.

### Setup Steps
1. Create app + bot at [Discord Developer Portal](https://discord.com/developers/applications)
2. Enable **Privileged Gateway Intents**: Message Content, Server Members
3. Copy bot token
4. Generate OAuth2 invite URL with `bot` + `applications.commands` scopes
5. Set token on gateway (never in chat):

```bash
openclaw config set channels.discord.token '"YOUR_BOT_TOKEN"' --json
openclaw config set channels.discord.enabled true --json
```

### Config

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "...",
      dmPolicy: "pairing",         // or "allowlist"
      allowFrom: ["USER_ID"],       // Discord user IDs
      groupPolicy: "allowlist",
      groupAllowFrom: ["GUILD_ID"],
    }
  }
}
```

### Features
- Slash commands via `applications.commands` scope
- Thread-bound ACP sessions (`sessions_spawn` with `thread: true`)
- Reactions, embeds, file attachments
- `message` tool supports: `action=send/edit/delete/poll/react/topic-create`
- Suppress link embeds by wrapping URLs in `<>`: `<https://example.com>`

### Discord-specific Notes
- No markdown tables! Use bullet lists
- Wrap multiple links in `<>` to suppress embeds
- Thread spawns: use `sessions_spawn(runtime:"acp", thread:true)` — don't manually create threads

---

## WhatsApp

**Method:** WhatsApp Web protocol via Baileys library (unofficial)
**Status:** Production-ready. Gateway owns the linked session.

### Setup Steps

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15551234567"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    }
  }
}
```

```bash
openclaw channels login --channel whatsapp   # shows QR code, scan with WhatsApp
openclaw gateway                              # start gateway
openclaw pairing list whatsapp               # see pending requests
openclaw pairing approve whatsapp <CODE>     # approve first message
```

### Key Points
- Uses WhatsApp Web protocol — **not the official business API**
- Recommended: use a **separate number** for the bot
- Links to your WhatsApp account (one linked device slot)
- If WhatsApp detects automation: possible ban risk (use a dedicated number)
- Multi-account: `--account work` flag for separate sessions

### Formatting Notes
- No markdown headers
- Use **bold** or CAPS for emphasis
- No tables — use bullet lists

---

## Signal

**Method:** `signal-cli` (external Java CLI over HTTP JSON-RPC + SSE)
**Status:** Supported. Requires separate setup.

### Prerequisites
- `signal-cli` installed (Java required for JVM build)
- A phone number that can receive SMS for registration
- OR: link to existing Signal account via QR

### Setup Options

**Option A — Link existing account:**
```bash
signal-cli link -n "OpenClaw"   # scan QR with Signal app
```

**Option B — Register new number:**
```bash
signal-cli register +15551234567 --captcha <token>
signal-cli verify +15551234567 --verification-code 123456
```

### Config

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",    // bot's Signal number
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    }
  }
}
```

### Key Points
- Gateway talks to `signal-cli` over HTTP (not embedded libsignal)
- Use a **separate bot number** — gateway ignores messages from the bot's own number (loop protection)
- DMs → main session; Groups → isolated session per group
- Supports `/config set` commands (disable with `configWrites: false`)

---

## Common Pattern Across All Channels

```
Install/Auth → Configure access policy → Start gateway → Approve first pairing
```

| Config key | What it controls |
|------------|-----------------|
| `dmPolicy` | Who can DM: `"pairing"` \| `"allowlist"` \| `"open"` |
| `allowFrom` | Allowlisted sender IDs/numbers |
| `groupPolicy` | Group access: `"allowlist"` \| `"open"` \| `"off"` |
| `groupAllowFrom` | Allowlisted group IDs |

**Pairing mode:** Unknown senders get a code. You approve with `openclaw pairing approve <channel> <CODE>`. Expires in 1 hour, max 3 pending.
