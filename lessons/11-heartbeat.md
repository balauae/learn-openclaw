# 11 — Heartbeat

## What It Is

A periodic agent turn that fires every N minutes. The agent checks if anything needs attention and either alerts you or stays silent.

## The Contract

- Nothing urgent → reply `HEARTBEAT_OK` → silently dropped, you never see it
- Something needs attention → reply with alert text → delivered to you

## HEARTBEAT.md

Optional checklist file in the workspace. Injected into every heartbeat prompt. Keep it small — it runs every 30 minutes.

```markdown
# Heartbeat checklist
- Check GPU temp (nvidia-smi)
- Any urgent emails?
- Calendar events in next 2h?
```

If the file is empty (only headers/blanks), heartbeat is skipped entirely — saves tokens.

## Config

```json5
heartbeat: {
  every: "30m",          // interval (0m = disabled)
  target: "last",        // "last" | "none" | "telegram"
  activeHours: {
    start: "08:00",
    end: "22:00",
    timezone: "Asia/Dubai"
  },
  lightContext: true,    // only inject HEARTBEAT.md, skip rest of workspace
  model: "mistral",      // cheaper model for routine checks
}
```

## target: "last" vs "none"

- `"none"` — heartbeat runs silently, alerts only visible if you're actively chatting
- `"last"` — alerts get **pushed** to your last active channel (e.g. Telegram)

## HEARTBEAT_OK Stripping

`HEARTBEAT_OK` is stripped if at the start/end of a reply with ≤300 chars remaining. In the **middle** of a reply — not treated specially, full message delivered.

## Manual Wake

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
```

## Cost Note

Each heartbeat = a full model turn (~48/day at 30min intervals). Use `lightContext: true` and a cheap model to keep costs down.
