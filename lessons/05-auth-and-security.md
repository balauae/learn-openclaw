# 05 — Authentication & Security

## API Keys

Stored in config under `models.providers`:
```json5
models: {
  providers: {
    anthropic: { apiKey: "sk-ant-..." }
  }
}
```

Or via environment variables (`ANTHROPIC_API_KEY`, etc.). Keys are never logged.

## Allowlists

Restrict who can interact with the bot:
```json5
channels: {
  telegram: {
    dmPolicy: "allowlist",
    allowlist: ["8523037700"]
  }
}
```

Policies: `allow` | `deny` | `allowlist`

## dmPolicy vs groupPolicy

- `dmPolicy` — controls direct/private messages
- `groupPolicy` — controls group chat access

Set either to `deny` to block that surface entirely.

## Elevated Mode

Certain tools (e.g. host-level exec) require elevated permissions. Users must be in an elevated allowlist and explicitly request it. Keeps destructive actions behind an extra gate.

## Sandbox

Exec commands can run in a sandboxed environment with restricted filesystem/network access. Configured per-session or globally via `security.sandbox`.
