# 21 — CLI & Commands

## Overview

The `openclaw` CLI is your control panel for everything. All subcommands follow:

```bash
openclaw [--dev] [--profile <name>] <command> [options]
```

Global flags:
- `--dev` — isolate state under `~/.openclaw-dev` (safe sandbox for testing)
- `--profile <name>` — isolate state under `~/.openclaw-<name>`
- `--json` — machine-readable output
- `--no-color` — disable ANSI colors
- `-V` / `--version` — print version

---

## Command Groups

### 🚀 Setup & Onboarding

| Command | What it does |
|---------|-------------|
| `openclaw setup` | First-time install wizard |
| `openclaw onboard` | Interactive onboarding flow |
| `openclaw configure` | Guided config editor |
| `openclaw doctor` | Health check — finds and fixes config issues |
| `openclaw update` | Update OpenClaw to latest version |
| `openclaw reset` | Factory reset (destructive!) |
| `openclaw uninstall` | Remove OpenClaw |

### ⚙️ Config

| Command | What it does |
|---------|-------------|
| `openclaw config get [path]` | Read config value |
| `openclaw config set <path> <value> --json` | Write config value |
| `openclaw config unset <path>` | Remove config key |

```bash
# Examples
openclaw config get channels.telegram.dmPolicy
openclaw config set agents.defaults.model '"anthropic/claude-sonnet-4-6"' --json
openclaw config set channels.discord.enabled true --json
```

### 🌐 Gateway

| Command | What it does |
|---------|-------------|
| `openclaw gateway` | Start gateway (foreground) |
| `openclaw gateway status` | Show running status |
| `openclaw gateway start` | Start as daemon |
| `openclaw gateway stop` | Stop daemon |
| `openclaw gateway restart` | Restart daemon |
| `openclaw logs` | Stream gateway logs |
| `openclaw status` | Session + model status |
| `openclaw health` | Health check endpoint |

### 💬 Channels

| Command | What it does |
|---------|-------------|
| `openclaw channels list` | List configured channels |
| `openclaw channels status` | Connection status per channel |
| `openclaw channels login --channel <name>` | Authenticate (QR, token, etc.) |
| `openclaw channels logout --channel <name>` | Disconnect channel |
| `openclaw pairing list <channel>` | Pending pairing requests |
| `openclaw pairing approve <channel> <code>` | Approve a pairing request |
| `openclaw pairing reject <channel> <code>` | Reject a pairing request |
| `openclaw qr` | Show QR code for pairing |

### 🤖 Agents & Sessions

| Command | What it does |
|---------|-------------|
| `openclaw agent list` | List agents |
| `openclaw agents list` | Alias |
| `openclaw sessions` | List active sessions |
| `openclaw acp` | ACP harness commands |
| `openclaw tui` | Terminal UI connected to gateway |
| `openclaw message` | Send a message from CLI |

```bash
# Open TUI
openclaw tui

# Send a message to a session
openclaw message --channel telegram --to +1234567890 "Hello"
```

### 🔒 Security & Secrets

| Command | What it does |
|---------|-------------|
| `openclaw security audit` | Security posture audit |
| `openclaw secrets reload` | Reload secrets from disk |
| `openclaw secrets migrate` | Migrate secrets format |
| `openclaw approvals` | Manage pending tool approvals |

### 🐳 Sandbox

| Command | What it does |
|---------|-------------|
| `openclaw sandbox explain` | Show effective sandbox config |
| `openclaw sandbox explain --agent work` | Per-agent sandbox status |
| `openclaw sandbox explain --json` | Machine-readable output |

### 🌍 Nodes (ClawNet)

| Command | What it does |
|---------|-------------|
| `openclaw nodes list` | List paired nodes |
| `openclaw nodes describe --node <id>` | Node details |
| `openclaw nodes pending` | Pending pairing requests |
| `openclaw nodes approve <requestId>` | Approve node pairing |
| `openclaw nodes notify --node <id> --body "text"` | Send notification |
| `openclaw nodes run --node <id> <command>` | Run command on node |
| `openclaw nodes camera snap --node <id>` | Take photo |
| `openclaw nodes location get --node <id>` | Get GPS location |

### 🌐 Browser

| Command | What it does |
|---------|-------------|
| `openclaw browser status` | Browser running status |
| `openclaw browser start` | Start browser |
| `openclaw browser stop` | Stop browser |
| `openclaw browser open <url>` | Open URL |
| `openclaw browser snapshot` | Get accessibility tree |
| `openclaw browser screenshot` | Take screenshot |
| `openclaw browser navigate <url>` | Navigate current tab |
| `openclaw browser click <ref>` | Click element |
| `openclaw browser type <ref> <text>` | Type into element |
| `openclaw browser evaluate --fn <code>` | Run JS |
| `openclaw browser tabs` | List open tabs |

### ⏰ Cron

| Command | What it does |
|---------|-------------|
| `openclaw cron list` | List scheduled jobs |
| `openclaw cron add` | Add a job |
| `openclaw cron remove <id>` | Remove a job |
| `openclaw cron run <id>` | Trigger job immediately |
| `openclaw cron status` | Scheduler status |

### 🧠 Memory & Skills

| Command | What it does |
|---------|-------------|
| `openclaw memory` | Memory management |
| `openclaw skills list` | List loaded skills |
| `openclaw skills reload` | Reload skills |

### 🔌 Plugins

| Command | What it does |
|---------|-------------|
| `openclaw plugins list` | List installed plugins |
| `openclaw plugins install <spec>` | Install plugin |
| `openclaw plugins remove <id>` | Remove plugin |

### 🛠️ Utilities

| Command | What it does |
|---------|-------------|
| `openclaw doctor` | Diagnose + fix issues |
| `openclaw dashboard` | Open web dashboard |
| `openclaw backup create` | Create config backup |
| `openclaw backup verify` | Verify backup |
| `openclaw models` | List available models |
| `openclaw docs [query]` | Search docs |
| `openclaw system` | System info |
| `openclaw dns` | DNS/network diagnostics |
| `openclaw completion` | Shell completion setup |

---

## Most Useful Day-to-Day

```bash
openclaw gateway status       # is it running?
openclaw doctor               # anything broken?
openclaw logs                 # what's happening?
openclaw status               # session info
openclaw config get           # read config
openclaw sandbox explain      # debug sandbox
openclaw tui                  # interactive terminal UI
```

---

## That's All 21 Topics! 🎉

Full OpenClaw architecture covered:
- Week 1: Gateway → Config → Sessions → Telegram → Auth → Tools → Exec
- Week 2: Skills → Sub-agents → Memory → Heartbeat → Cron → Thinking → Streaming → Nodes → Browser
- Week 3: ComfyUI → Docker → Channels → Plugins → CLI
