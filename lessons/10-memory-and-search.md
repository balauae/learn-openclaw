# 10 — Memory & Search

## The Core Idea

Memory is just **plain Markdown files on disk**. Nothing survives a session restart unless it's written to a file.

## Two Layers

| File | Purpose |
|---|---|
| `memory/YYYY-MM-DD.md` | Daily log — raw notes, append-only |
| `MEMORY.md` | Long-term curated memory — loaded in private sessions only |

## Two Tools

- **`memory_search`** — semantic search across all memory files, returns relevant snippets
- **`memory_get`** — read a specific file or line range

## How Search Works

Files are chunked (~400 tokens each) and embedded into a SQLite vector index at `~/.openclaw/memory/main.sqlite`. Search finds semantically similar chunks even if the wording differs.

## Hybrid Search (BM25 + Vector)

Combines:
- **Vector**: great for "means the same thing"
- **BM25**: great for exact tokens (IDs, error strings, variable names)

Result: `score = 0.7 × vector + 0.3 × keyword`

## Power Features (off by default)

**MMR re-ranking** — removes near-duplicate results when multiple daily notes say the same thing.

**Temporal decay** — recent memories rank higher. Old notes fade:
- Today: 100% score
- 30 days ago: 50%
- 6 months ago: ~1.6%
- `MEMORY.md` and non-dated files are **never** decayed.

Enable both:
```json5
memorySearch: {
  query: {
    hybrid: {
      enabled: true,
      mmr: { enabled: true },
      temporalDecay: { enabled: true, halfLifeDays: 30 }
    }
  }
}
```

## Auto Memory Flush

When a session nears the context limit, OpenClaw silently triggers a "write your memories now" agent turn before compaction. You never see it.

## Embedding Providers

Auto-selected in order: local GGUF → OpenAI → Gemini → Voyage → Mistral → Ollama (manual only)
