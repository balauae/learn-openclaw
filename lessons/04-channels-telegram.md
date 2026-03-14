# 04 — Channels (Telegram)

> How the Telegram plugin works end-to-end.

## Architecture

```
Telegram servers
      ↕ (polling or webhook)
OpenClaw Gateway (Telegram plugin)
      ↕
Agent session
      ↕
Claude / model
```

The Telegram plugin maintains a bot connection (via `node-telegram-bot-api` or similar), receives updates, routes them to the correct agent session, and sends replies back.

## Config

```json5
channels: {
  telegram: {
    accounts: {
      default: {
        botToken: "YOUR_BOT_TOKEN"   // from BotFather
      }
    }
  }
}
```

Multiple bots = multiple account entries.

## Message Routing

Inbound messages are matched to sessions by `chat_id` (and optionally `message_thread_id` for topics/forums). Each unique chat gets its own session.

## Policies

Control who can talk to your bot:

```json5
channels: {
  telegram: {
    dmPolicy: "allowlist",       // only allowlisted users in DMs
    groupPolicy: "deny",         // no group chats
    allowlist: ["8523037700"]    // your Telegram user ID
  }
}
```

Policies: `allow` | `deny` | `allowlist`

## Reactions

OpenClaw supports emoji reactions on Telegram in two modes:
- `minimal` — only when genuinely relevant
- `all` — react to everything

Configure: `channels.telegram.reactions: { mode: "minimal" }`

## Inline Buttons

Replies can include inline keyboard buttons:
```json
"buttons": [[{"text": "Yes", "callback_data": "yes"}, {"text": "No", "callback_data": "no"}]]
```

## Topics / Forum Threads

Telegram supergroups with topics use `message_thread_id`. Target a specific topic in cron delivery:
```
-1001234567890:topic:42
```

## Delivery Chunking

Long replies are automatically chunked to respect Telegram's 4096-character message limit. Markdown formatting is applied per-chunk.

## Multi-Account

Run multiple bots from one Gateway:
```json5
channels: {
  telegram: {
    accounts: {
      "personal-bot": { botToken: "TOKEN_1" },
      "ops-bot": { botToken: "TOKEN_2" }
    }
  }
}
```
