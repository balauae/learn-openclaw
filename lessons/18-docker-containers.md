# 18 — Docker & Containers

## Two Distinct Roles

Docker in OpenClaw serves **two completely separate purposes**:

| Role | What it is | Required? |
|------|------------|-----------|
| **Containerized Gateway** | Run OpenClaw itself inside Docker | ❌ Optional |
| **Agent Sandbox** | Run AI tool execution inside Docker | ❌ Optional |

These are independent — you can use either, both, or neither.

## Agent Sandbox (The Main Feature)

By default, tools like `exec`, `read`, `write` run directly on the host. The sandbox moves tool execution into a Docker container to limit blast radius.

```
Gateway (host) → tool call → Docker container → result back
                              (isolated FS, no network)
```

**Key point: The Gateway stays on the host. Only tools move into Docker.**

### Configuration

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",        // "off" | "non-main" | "all"
        scope: "session",        // "session" | "agent" | "shared"
        workspaceAccess: "none"  // "none" | "ro" | "rw"
      }
    }
  }
}
```

### `mode` — When to sandbox

| Value | Behavior |
|-------|---------|
| `"off"` | No Docker, everything on host |
| `"non-main"` | Sandbox sub-agents + group chats only |
| `"all"` | Everything sandboxed |

### `scope` — Container lifecycle

| Value | Behavior |
|-------|---------|
| `"session"` | One container per session (most isolated) |
| `"agent"` | One container per agent type |
| `"shared"` | One container for everything |

### `workspaceAccess` — What the sandbox can see

| Value | Behavior |
|-------|---------|
| `"none"` | Sandbox gets its own empty workspace under `~/.openclaw/sandboxes/` |
| `"ro"` | Workspace mounted read-only at `/agent` |
| `"rw"` | Workspace mounted read-write at `/workspace` |

## Sandbox Images

Three images, built once:

```bash
scripts/sandbox-setup.sh          # Minimal: openclaw-sandbox:bookworm-slim
scripts/sandbox-common-setup.sh   # With curl, jq, python3, git, node
scripts/sandbox-browser-setup.sh  # For sandboxed browser tool
```

Default image has **no network** (`docker.network: "none"`) — sandboxed containers can't make outbound calls.

## The Three Control Layers

```
1. Tool Policy (allow/deny)   ← "Can this tool exist at all?"
         ↓
2. Sandbox mode               ← "Where does it run? Host or Docker?"
         ↓
3. Elevated exec              ← "Can this exec escape to host?"
```

- **Tool policy** always wins first — if `exec` is denied, sandbox doesn't matter
- **Elevated** (`tools.elevated`) is the escape hatch: lets specific `exec` calls run on host even when sandboxed

## Custom Bind Mounts

Pierce the sandbox filesystem for specific paths:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: [
            "/home/bala/dev/source:/source:ro",
            "/var/data:/data:rw"
          ]
        }
      }
    }
  }
}
```

**Blocked by OpenClaw:** `docker.sock`, `/etc`, `/proc`, `/sys`, `/dev` — can't accidentally mount dangerous paths.

## Debug Command

```bash
openclaw sandbox explain
openclaw sandbox explain --agent work --json
```

Shows: effective mode, why something is blocked, fix-it config key path.

## Containerized Gateway (Optional)

Run OpenClaw *itself* inside Docker:

```bash
./docker-setup.sh

# With sandbox enabled:
OPENCLAW_SANDBOX=1 ./docker-setup.sh
```

Access at `http://127.0.0.1:18789/`. Not useful for local dev — native is faster.

## Minimal Enable Example

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none"
      }
    }
  }
}
```

## When to Use Sandbox

Use it when you want to give agents more autonomy with less risk — e.g. letting a sub-agent run arbitrary code without touching your actual home directory.
