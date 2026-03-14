# 06 — Tools System

## How Tools Are Exposed

OpenClaw builds the tool list dynamically per session based on:
1. Built-in tools (exec, read, write, browser, etc.)
2. Plugin tools (comfyui, lobster, etc.)
3. Policy filters (what's allowed for this session/channel)

The model only sees tools that pass the policy check.

## Tool Policies

Control which tools are available:
```json5
tools: {
  deny: ["exec"],           // block specific tools
  alsoAllow: ["lobster"],   // add optional tools
}
```

Per-channel overrides are also supported.

## Elevated Tools

Some tools are gated behind elevated mode (e.g. host-level exec, filesystem writes outside workspace). The user must be on an elevated allowlist.

## Key Built-in Tools

| Tool | What it does |
|---|---|
| `exec` | Run shell commands |
| `Read` / `Write` / `Edit` | File operations |
| `browser` | Web browser automation |
| `memory_search` / `memory_get` | Search memory files |
| `cron` | Manage scheduled jobs |
| `message` | Send channel messages |
| `web_search` / `web_fetch` | Internet access |
| `sessions_spawn` | Launch sub-agents |
| `image` | Analyze images |
| `tts` | Text to speech |

## Tool Call Style

Tools are invoked as structured JSON by the model. OpenClaw handles the actual execution and returns results back into the context.
