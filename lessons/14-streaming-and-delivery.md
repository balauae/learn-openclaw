# 14 — Streaming & Delivery

## Two Separate Layers

OpenClaw has two independent streaming systems that often get confused:

| Layer | What it is |
|---|---|
| **Block streaming** | Sends completed chunks as real channel messages while generating |
| **Preview streaming** | Updates a temporary "typing..." message live while generating |

There is **no true token-delta streaming** — it's message-based (send + edits).

---

## Block Streaming

Sends the reply in coarse chunks as they're ready, rather than waiting for the full response.

```
off (default) → wait for full reply, send once
on            → send chunks as they complete
```

Enable globally:
```json5
agents: {
  defaults: {
    blockStreamingDefault: "on",
    blockStreamingBreak: "text_end"   // or "message_end"
  }
}
```

### text_end vs message_end

- **`text_end`** — emit chunks as soon as they're ready (most "live" feeling)
- **`message_end`** — buffer everything, flush at end (still chunked if too long)

### Chunking Algorithm

Chunks respect natural break points:
`paragraph → newline → sentence → whitespace → hard break`

Code fences are **never** split mid-block. If forced to split at max length, it closes and reopens the fence to keep Markdown valid.

Config:
```json5
blockStreamingChunk: {
  minChars: 200,   // don't emit until buffer is this big
  maxChars: 1000   // split here at the latest
}
```

### Coalescing

Merges multiple small consecutive chunks before sending — avoids single-line spam:
```json5
blockStreamingCoalesce: {
  idleMs: 500,    // wait this long for more content before flushing
  minChars: 1500, // default for Signal/Slack/Discord
  maxChars: 3000
}
```

### Human-like Pacing

Add a randomized pause between block messages to feel more natural:
```json5
agents: {
  defaults: {
    humanDelay: "natural"   // 800–2500ms between blocks
    // or: { minMs: 500, maxMs: 2000 }
  }
}
```

---

## Preview Streaming (Telegram/Discord/Slack)

Updates a live "draft" message while the model is generating. The final answer replaces it.

Configured per channel: `channels.<channel>.streaming`

| Mode | Behaviour |
|---|---|
| `off` | No preview — reply appears only when done |
| `partial` | Single preview message, updated with latest text |
| `block` | Preview updated in appended steps |
| `progress` | Status indicator during generation, final answer at end |

### Telegram specifically

- Uses `sendMessage` + `editMessageText` for live updates
- `/reasoning stream` writes thinking into the preview bubble, then sends clean final answer
- Preview streaming is **skipped** if block streaming is also enabled (avoids double-streaming)

### Discord

- Send + edit for preview
- `block` mode uses draft chunking

### Slack

- `partial` can use Slack's native streaming API (`chat.startStream`)
- `progress` mode is Slack-only

---

## Channel Text Limits

Each channel has a hard cap on message length. OpenClaw auto-chunks to stay under:

| Channel | Limit |
|---|---|
| Telegram | 4096 chars |
| Discord | 2000 chars (+ `maxLinesPerMessage: 17` default) |
| WhatsApp | Configurable via `textChunkLimit` |

Long replies are split at natural break points — never mid-sentence or mid-code-block.

---

## Quick Reference

| Goal | Config |
|---|---|
| Live chunked delivery on Telegram | `blockStreamingDefault: "on"`, `blockStreamingBreak: "text_end"` |
| Full reply, then send | `blockStreamingDefault: "off"` (default) |
| Live preview in Telegram | `channels.telegram.streaming: "partial"` |
| See reasoning live | `/reasoning stream` |
| Natural pacing between chunks | `humanDelay: "natural"` |
