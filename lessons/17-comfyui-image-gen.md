# 17 — ComfyUI / Image Gen

## What It Is

OpenClaw generates images locally using **ComfyUI** (a node-based Stable Diffusion UI) running at `localhost:8188`. The integration is delivered as a **skill** (`~/.openclaw/skills/comfyui/`).

Model in use: **Z-Image-Turbo** — a fast distilled diffusion model, only needs 8 steps.

## The Full Pipeline

```
You (Telegram) → OpenClaw → Skill (SKILL.md) → Python script → ComfyUI API → GPU → PNG → back to you
```

## How the Skill Triggers

- `SKILL.md` description tells OpenClaw when to use it: *"generate, create, draw, or make an image..."*
- Skills are auto-discovered at session start from `~/.openclaw/skills/`
- When triggered, OpenClaw reads `SKILL.md` and follows the instructions

## The Python Script

`scripts/comfyui_generate.py` handles everything:

```bash
python3 comfyui_generate.py \
  --prompt "a cat in space" \
  --width 1024 --height 1024 \
  --steps 8 \
  --output /tmp/generated.png
```

### Parameters

| Flag | Default | Notes |
|------|---------|-------|
| `--prompt` | required | Text description |
| `--width/height` | 1024 | 512–2048px supported |
| `--steps` | 8 | 4–12 sweet spot |
| `--seed` | random | Fix for reproducible results |
| `--output` | `/tmp/openclaw-comfyui-<id>.png` | Output path |
| `--timeout` | 120s | Increase for large images |

### Output

```json
{"ok": true, "path": "/tmp/openclaw-comfyui-abc123.png", "filename": "...", "prompt_id": "...", "size": 123456}
```

## ComfyUI Workflow Graph (Under the Hood)

ComfyUI uses a **node graph API** — you submit a full JSON graph, not just a prompt.

Nodes used by Z-Image-Turbo workflow:

| Node | Purpose |
|------|---------|
| `UNETLoader` | Loads `z_image_turbo_bf16.safetensors` |
| `CLIPLoader` | Loads `qwen_3_4b.safetensors` (Qwen 3 4B text encoder) |
| `VAELoader` | Loads `ae.safetensors` for latent ↔ pixel conversion |
| `CLIPTextEncode` | Encodes your prompt into conditioning |
| `ConditioningZeroOut` | Creates empty negative conditioning |
| `EmptySD3LatentImage` | Blank latent canvas at target resolution |
| `ModelSamplingAuraFlow` | AuraFlow sampler wrapper (shift=3) |
| `KSampler` | Runs the denoising loop for N steps |

## API Flow

1. **POST `/prompt`** — submit workflow graph + `client_id` + `prompt_id`
2. **Poll `/history/{prompt_id}`** — check every second until `completed: true`
3. **GET `/view?filename=...`** — download the raw PNG bytes
4. Save to `/tmp/`, send via `message` tool to Telegram

## Delivery

After the script outputs the path, OpenClaw sends it to Telegram:
```
message(action="send", media="/tmp/openclaw-comfyui-abc.png")
```

## Gotchas

- ComfyUI uses ~20GB VRAM while generating
- No network required — fully local
- Output stays in `/tmp/` — not auto-saved anywhere persistent
- If ComfyUI isn't running → immediate connection error
- Set `COMFYUI_HOST` env var to point at a different host/port

## Prompt Tips

- Be descriptive: subject + style + lighting + mood
- 8 steps is usually enough; try 10–12 for higher quality
- 1024×1024 is the sweet spot
