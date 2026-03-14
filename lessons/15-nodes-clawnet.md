# 15 — Nodes (ClawNet)

## What Is a Node?

A **node** is a companion device (phone, Mac, headless Linux box) that connects to the Gateway and exposes hardware capabilities — camera, screen, location, notifications, shell exec — to the agent.

The Gateway is the brain. Nodes are its eyes, ears, and hands.

```
Agent (Gateway) ──WebSocket──► Node (iOS/Android/Mac/Linux)
                                 ├── camera
                                 ├── screen recording
                                 ├── location
                                 ├── notifications
                                 └── system.run (exec)
```

## Pairing

Nodes connect to the Gateway WebSocket and present a device identity. You approve them:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw nodes status          # see all paired nodes
openclaw nodes describe --node <name>
```

## What Nodes Can Do

| Capability | Platforms | Command |
|---|---|---|
| Camera photo | iOS, Android, macOS | `camera.snap` |
| Camera video clip | iOS, Android, macOS | `camera.clip` |
| Screen recording | macOS, some Android | `screen.record` |
| Canvas (WebView) | iOS, Android, macOS | `canvas.*` |
| Location | iOS, Android | `location.get` |
| Notifications | Android | `notifications.list` |
| Shell exec | macOS, headless Linux/Windows | `system.run` |
| SMS | Android | `sms.send` |

## Camera

```bash
# Take a photo (both front + back by default)
openclaw nodes camera snap --node my-phone

# Specific camera
openclaw nodes camera snap --node my-phone --facing front

# Video clip (10 seconds)
openclaw nodes camera clip --node my-phone --duration 10s

# No audio
openclaw nodes camera clip --node my-phone --duration 5s --no-audio
```

Photos are capped at 5MB (recompressed). Clips capped at 60s.

**Foreground only** — node app must be in foreground, otherwise returns `NODE_BACKGROUND_UNAVAILABLE`.

## Screen Recording

```bash
openclaw nodes screen record --node my-mac --duration 10s --fps 15
```

Requires Screen Recording permission on macOS.

## Location

```bash
openclaw nodes location get --node my-phone
openclaw nodes location get --node my-phone --accuracy precise
```

Off by default — must enable in node app settings.

## Remote Exec (Node Host)

The most powerful feature. Run shell commands on a remote machine via the agent:

```bash
# Start a headless node host on the remote machine
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Server"

# Then exec routes to that machine
exec({ command: "docker ps", host: "node", node: "Build Server" })
```

### As a Service

```bash
openclaw node install --host <gateway-host> --port 18789
openclaw node restart
```

### Allowlist Commands

```bash
openclaw approvals allowlist add --node "Build Server" "/usr/bin/docker"
```

## Canvas (WebView Control)

Nodes can display a WebView that the agent can control:

```bash
openclaw nodes canvas present --node my-phone --target https://example.com
openclaw nodes canvas snapshot --node my-phone --format png   # screenshot
openclaw nodes canvas eval --node my-phone --js "document.title"
openclaw nodes canvas hide --node my-phone
```

## Notifications (Android)

```bash
openclaw nodes invoke --node my-phone --command notifications.list --params '{}'
```

## Push Notifications to Node

```bash
openclaw nodes notify --node my-mac --title "Done" --body "Build finished"
```

## Practical Setup

For your rig (bala-pc), the machine itself can be a node host — useful if you ever want a second machine or want to exec commands routed through the agent cleanly. Start with:

```bash
openclaw node run --host 127.0.0.1 --port 18789 --display-name "bala-pc"
```
