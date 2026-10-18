# 06 — Implementation Roadmap

## Phase 1: Core Storage & Domain Engine (`src/core/`)
- [ ] Initialize TypeScript project (`package.json`, `tsconfig.json`, build/dev scripts).
- [ ] Implement `VaultSyncService` to read/write Markdown files with YAML frontmatter (`gray-matter`) in `vault/{sources,notes,actions,harness}/` and `vault/profile.md`.
- [ ] Set up `better-sqlite3` database schema (`sources`, `notes`, `action_points`, `harness_applications`), FTS5 virtual table (`knowledge_fts`), and `sqlite-vec` vector index (`knowledge_vec`).
- [ ] Implement Hybrid Search Engine (FTS5 BM25 + Vector Similarity with Reciprocal Rank Fusion).

## Phase 2: Source Ingestion & LLM Analysis Pipeline (`src/core/ingest/`)
- [ ] Build source extractors:
  - Web URL parser (`@mozilla/readability` + `turndown` + Jina fallback)
  - YouTube transcript extractor (`youtube-transcript` + metadata)
  - Local file parser (`.md`, `.txt`, `.pdf` via `unpdf`, `.docx` via `mammoth`)
  - Raw text/Markdown handler
- [ ] Implement `AnalysisService` using Vercel AI SDK (`generateObject` + `ExtractionResultSchema`) with `vault/profile.md` context injection.
- [ ] Wire end-to-end ingestion -> LLM extraction -> auto-save Markdown -> SQLite + Embedding index.

## Phase 3: MCP Server (`src/mcp/`)
- [ ] Implement MCP server using `@modelcontextprotocol/sdk` (`stdio` and HTTP/SSE transports).
- [ ] Register core tools:
  - `advise_on_workflow` (Context/Workflow Advisor)
  - `list_harness_applications` & `update_harness_status` (Harness Generator)
  - `search_knowledge` & `get_knowledge_item` (Knowledge Search)
  - `list_action_points` & `update_action_point` (Action Tracker)
  - `ingest_source` & `update_user_profile` (Headless Ingestion & Profile)
- [ ] Register MCP Prompts (`review-my-workflow`, `upgrade-my-harness`) and Resources (`knowledge://profile`, `knowledge://actions/todo`, `knowledge://harness/proposed`).

## Phase 4: HTTP API & Web UI Dashboard (`src/server/` & `src/web/`)
- [ ] Build REST API in Hono (`src/server/`) exposing ingestion, search, CRUD, and profile endpoints.
- [ ] Scaffold React + Vite + Tailwind CSS Web UI (`src/web/`).
- [ ] Build Universal Ingest Bar (URL, YouTube, file drop, raw text) with live status feedback.
- [ ] Build Sources Explorer, Notes Browser, Action Points Board, AI Harness Lab, and Profile/MCP Setup screen.
