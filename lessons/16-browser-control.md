# 16 — Browser Control

## What It Is

OpenClaw can control a real browser — open tabs, take screenshots, click, type, scrape pages, fill forms. Powered by Chrome DevTools Protocol (CDP).

## Two Profiles

| Profile | What it is |
|---|---|
| `openclaw` | Isolated, agent-managed browser (separate from your daily Chrome) |
| `chrome` | Extension relay — controls your existing Chrome via the OpenClaw browser extension |

`openclaw` profile = safe sandbox. `chrome` profile = your actual browser (use with care).

## Quick Start

```bash
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
openclaw browser --browser-profile openclaw status
```

## Config

```json5
browser: {
  enabled: true,
  defaultProfile: "openclaw",   // or "chrome" for extension relay
  headless: false,              // true = invisible browser
  executablePath: "/usr/bin/google-chrome",  // override auto-detect
  profiles: {
    openclaw: { cdpPort: 18800 },
    work: { cdpPort: 18801 },
  }
}
```

Auto-detects: system default browser → Chrome → Brave → Edge → Chromium

## Agent Tool: `browser`

The `browser` tool gives the agent full control:

```
action: status | start | stop | open | snapshot | screenshot | navigate | act | tabs | close
```

### Snapshot vs Screenshot

- **`snapshot`** — accessibility tree (text-based, fast, used for automation)
- **`screenshot`** — actual PNG/JPEG image of the page

Snapshots are preferred for automation (cheaper, faster). Screenshots when you need to actually see the page visually.

### Actions (clicking, typing, etc.)

```json5
browser({
  action: "act",
  profile: "openclaw",
  request: {
    kind: "click",
    ref: "e12"          // element ref from snapshot
  }
})
```

Action kinds: `click` | `type` | `press` | `hover` | `drag` | `select` | `fill` | `evaluate`

### Refs

Snapshots return element refs (e.g. `e12`). Use the same `targetId` across calls to keep working on the same tab.

Two ref modes:
- `refs: "role"` (default) — role+name based
- `refs: "aria"` — stable Playwright aria-ref IDs (better for multi-step automation)

## Chrome Extension Relay

For `profile: "chrome"` — requires the **OpenClaw Browser Relay** extension installed in Chrome. User clicks the toolbar icon to attach a tab (badge turns ON). Then the agent controls that tab.

Useful when you need the agent to act in your logged-in browser session (e.g. fill a form on a site you're already signed into).

## Remote Browser (CDP)

Point at a remote Chrome instance:
```json5
profiles: {
  remote: { cdpUrl: "http://192.168.1.42:9222" }
}
```

Useful for running the browser on a different machine (e.g. a headless server).

## Practical Uses

| Task | How |
|---|---|
| Scrape a page | `snapshot` → read the aria tree |
| Fill a form | `snapshot` → get refs → `act: fill` |
| Take a screenshot | `screenshot` |
| Navigate | `navigate` with URL |
| Run JS on page | `act: evaluate` with `fn: "document.title"` |
| Generate PDF | `action: pdf` |

## Security (SSRF)

By default, the browser can access private network addresses (trusted-network mode). Set `ssrfPolicy.dangerouslyAllowPrivateNetwork: false` to restrict to public URLs only.
