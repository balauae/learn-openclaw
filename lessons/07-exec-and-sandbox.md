# 07 — Exec & Sandbox

## The exec Tool

Runs shell commands on the host machine:
```json5
exec({ command: "nvidia-smi", timeout: 10 })
```

Supports:
- `pty: true` — pseudo-terminal for interactive CLIs
- `background: true` — fire and forget, poll later
- `yieldMs` — how long to wait before backgrounding
- `workdir` — working directory
- `elevated: true` — run with elevated permissions (if allowed)

## Background Execution

Long-running commands can be backgrounded:
```json5
exec({ command: "npm run build", background: true, yieldMs: 5000 })
// then poll with:
process({ action: "poll", sessionId: "...", timeout: 30000 })
```

## PTY Mode

Required for interactive tools (coding agents, ncurses UIs):
```json5
exec({ command: "htop", pty: true })
```

## Sandbox Modes

The `security` field on exec controls access level:
- `"deny"` — no exec allowed
- `"allowlist"` — only pre-approved commands
- `"full"` — unrestricted

## ask Mode

Controls whether user approval is needed before running:
- `"off"` — just run it
- `"on-miss"` — ask if not in allowlist
- `"always"` — always ask

## Host vs Sandbox Target

- `host: "sandbox"` — runs inside the OpenClaw sandbox container
- `host: "gateway"` — runs on the Gateway host machine
- `host: "node"` — runs on a paired ClawNet node
