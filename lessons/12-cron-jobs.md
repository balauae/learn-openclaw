# 12 — Cron Jobs

## What It Is

The Gateway's built-in scheduler. Persists jobs to disk, survives restarts, runs tasks at precise times.

## Two Execution Modes

| | Main Session | Isolated Session |
|---|---|---|
| `sessionTarget` | `"main"` | `"isolated"` |
| `payload.kind` | `"systemEvent"` | `"agentTurn"` |
| Context | Full chat history | Clean slate each run |
| Model | Main session model | Can override |
| Output | Routes through next heartbeat | Announces directly to channel |

These must match — mixing them breaks.

## Three Schedule Types

```json5
{ "kind": "at", "at": "2026-03-15T09:00:00Z" }     // one-shot
{ "kind": "every", "everyMs": 3600000 }              // fixed interval
{ "kind": "cron", "expr": "0 9 * * *", "tz": "Asia/Dubai" }  // cron expression
```

One-shot (`at`) jobs auto-delete after success by default.

## Delivery Modes (isolated jobs)

- `announce` — delivers to channel + brief summary to main session
- `webhook` — HTTP POST to a URL
- `none` — silent, nothing delivered

## CLI Examples

```bash
# One-shot reminder in 20 minutes
openclaw cron add --name "Reminder" --at "20m" \
  --session main --system-event "Check the build" --wake now --delete-after-run

# Daily 9am job
openclaw cron add --name "Morning brief" --cron "0 9 * * *" \
  --tz "Asia/Dubai" --session isolated \
  --message "Summarize today." --announce --channel telegram

# Heavy weekly analysis with a smarter model
openclaw cron add --name "Weekly review" --cron "0 9 * * 1" \
  --session isolated --message "Deep review..." --model opus --thinking high --announce
```

## Managing Jobs

```bash
openclaw cron list
openclaw cron run <jobId>         # trigger manually
openclaw cron runs --id <jobId>   # run history
openclaw cron edit <jobId>        # modify
openclaw cron remove <jobId>      # delete
```

## Retry Policy

- **Transient errors** (rate limits, network, 5xx): retry up to 3x with backoff (30s → 1m → 5m)
- **Permanent errors** (bad API key): disable immediately

## Heartbeat vs Cron

| Use | Mechanism |
|---|---|
| Batch periodic checks | Heartbeat |
| Exact time ("9am sharp") | Cron |
| One-shot reminder | Cron (`--at`) |
| Different model per task | Cron (isolated) |
| Context-aware decisions | Heartbeat |
