# 09 — Sub-agents

## What Are Sub-agents?

Isolated agent sessions spawned from the main session to handle tasks in parallel or in the background. They run independently and report back when done.

## sessions_spawn

```json5
sessions_spawn({
  task: "Refactor the auth module and write tests",
  runtime: "acp",        // "subagent" or "acp"
  mode: "run",           // "run" (one-shot) or "session" (persistent)
  agentId: "claude-code"
})
```

## Two Runtimes

| | `subagent` | `acp` |
|---|---|---|
| What | OpenClaw agent | External coding agent (Codex, Claude Code) |
| Use for | Orchestration, messaging, research | Coding tasks, file editing |
| PTY | No | Sometimes (Codex: yes, Claude Code: no) |

## run vs session

- `mode: "run"` — one-shot, completes and exits
- `mode: "session"` — persistent thread (good for Discord threads)

## Completion

Sub-agents are push-based — they announce when done. No need to poll.

## Managing Sub-agents

```json5
subagents({ action: "list" })   // see running sub-agents
subagents({ action: "steer", target: "...", message: "..." })  // send new instructions
subagents({ action: "kill", target: "..." })  // stop one
```

## When to Use

- Long coding tasks (don't block main session)
- Parallel research across multiple topics
- Thread-bound conversations on Discord
- Tasks needing a different model or context
