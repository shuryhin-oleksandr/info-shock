# 06 — Architecture

## Stack (proposed; library versions to be checked at implementation time)

| Concern | Choice | Why |
|---|---|---|
| Language | TypeScript (Node) | Decided |
| App | Next.js (web UI + HTTP API) | One process for UI and API |
| MCP | Official MCP TypeScript SDK; stdio + Streamable HTTP | Standard |
| LLM | Vercel AI SDK (or similar provider abstraction) | Any provider: Anthropic, OpenAI, Gemini, Ollama |
| DB | SQLite (file in the user's data dir) + vector extension (e.g. sqlite-vec) + FTS5 | Local, zero setup, single user |
| ORM | Drizzle or Prisma | Typed schema |
| Jobs | In-process job queue persisted in SQLite | No Redis needed locally |
| Extraction | Readability-style HTML extraction, PDF/DOCX parsers, YouTube transcript fetcher | Per source type |
| Transcription | Provider API (Whisper-class) or local whisper.cpp | Configurable |

## Components

```
           user's AI (Claude Code / Cursor / ...)
                 │  MCP (stdio or HTTP)        ▲ hook context
                 ▼                             │
┌──────────── local app ───────────────────────┴───────┐
│  MCP server ──┐                     Claude Code adapter (hooks → HTTP)
│  Web UI ──────┼──► Core services                      │
│  HTTP API ────┘     ├─ Ingestion (extract, chunk)     │
│                     ├─ Analyzer (notes/actions/harness)│
│                     ├─ Retrieval (FTS + vectors)      │
│                     ├─ Advisor (relevance → reasoning)│
│                     ├─ Learning (feedback, inference, │
│                     │   user model)                   │
│                     └─ Job queue                      │
│                 SQLite + files (uploads, transcripts) │
└──────────────────────────┬───────────────────────────┘
                           ▼
                 configured LLM provider(s)
```

## Advisor flow

1. Input: tool call (goal/plan/work summary) or hook event.
2. Retrieve: relevant notes/actions/harness items + user model + related suggestion history.
3. Relevance gate: cheap model or score threshold → may return `nothing_to_add`.
4. Reason: main model drafts suggestions with reasoning and evidence; drops ones the user already rejected (unless there's new evidence).
5. Store suggestions; return them in the standard format (03-advisor-mcp.md).

## Data model (entities)

- `sources` — type, url, title, metadata, content hash, status, error, raw/clean text location.
- `source_sections` — source id, position, text (for citations).
- `notes`, `actions`, `harness_applications` — fields from 02-ingestion.md, plus source id, citation, `edited_by_user`.
- `item_links` — related / duplicate / contradicts between items.
- `embeddings` — item id, kind, vector.
- `suggestions` — type, content, reasoning, confidence, evidence refs, session id, context.
- `suggestion_decisions` — suggestion id, decision, reason, usefulness, origin (explicit/inferred/ui), evidence, timestamp.
- `sessions` — client, project, goal, summary, outcome, transcript location.
- `user_profile` — structured JSON (goals, preferences, harness, patterns).
- `profile_changes` — diff, reason, origin session, reverted flag.
- `jobs` — type, payload, status, attempts.
- `settings` — providers, models per task, advice level.

## LLM usage per task

Each task has its own configurable model, so cheap models can do the cheap work:

| Task | Typical model size |
|---|---|
| Relevance check (continuous mode) | Small / fast |
| Embeddings | Embedding model (provider or local) |
| Source analysis | Large |
| Advising / critique | Large |
| Session inference, user model updates | Medium–large |
| Transcription | Speech-to-text model |

Prompts are versioned, so analysis can be re-run when a prompt changes.

## Privacy and safety

- All data in a local data directory. Nothing leaves the machine except calls to the configured providers.
- Transcripts and session data are redacted before storage and before LLM calls (API keys, tokens, `.env` contents, emails, by pattern).
- Option to use local models only (Ollama) for full privacy.
- The advisor never executes anything on the user's machine. It only returns text.
- Content from sources is untrusted: analysis prompts treat it as data, and advice is labeled as coming from the library, so a malicious article can't hijack the user's AI through the advisor.

## Packaging

- `npx`-style start command, or a desktop wrapper later.
- Data dir under the user's home (e.g. `~/.info-shock/`).
- Claude Code adapter shipped as a plugin or a hooks config snippet.
