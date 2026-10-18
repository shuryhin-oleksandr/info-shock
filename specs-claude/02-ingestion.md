# 02 — Ingestion and analysis

## Entry points

| Entry point | How |
|---|---|
| Web UI | Paste URL, paste text, upload file |
| MCP | `add_source` tool, called by the user's AI ("save this article to my library") |

Both go through the same pipeline. Ingestion is async: the caller gets a `source_id` and status right away.

## Source types

| Type | Input | Text extraction |
|---|---|---|
| Web article | URL | Fetch HTML → readability extraction → Markdown |
| Text / Markdown | Raw text | As is |
| PDF / document | File (PDF, DOCX, TXT, MD) | Text extraction per format |
| YouTube | URL | Official captions/transcript if present; otherwise audio → transcription |
| Podcast | URL to episode / audio file, or RSS item | Audio → transcription |

Transcription goes through the configured provider (e.g. Whisper API) or a local model. See open questions.

Deduplication: same canonical URL or same content hash → don't re-ingest; offer re-analysis instead.

## Pipeline

```
received → extracting → extracted → analyzing → analyzed
                    ↘ failed (with reason, retryable)
```

1. **Extract**: get clean text plus metadata (title, author, date, url, duration, language).
2. **Chunk**: split long text into sections for the model; keep section positions for citations.
3. **Analyze**: model pass(es) producing the three outputs below, plus a short summary and tags.
4. **Link**: embed new items; find related existing notes/actions; mark duplicates and conflicts ("contradicts note #42").
5. **Store**: save everything; make it searchable (full-text + vector).

Each output item keeps a **citation**: source id + section/quote, so advice can always point back to evidence.

The user can re-run analysis for a source (e.g. after a prompt or model change). Items the user edited by hand are kept, not overwritten.

## Outputs

### Note (main idea)

| Field | Notes |
|---|---|
| title | One line |
| body | The idea in the user's language, 1–5 sentences |
| quote | Supporting excerpt from the source |
| tags | Topics |
| importance | low / medium / high |
| related | Links to other notes |

### Action (what to do)

| Field | Notes |
|---|---|
| title | Imperative: "Add a pre-commit hook for ..." |
| why | What the user gains |
| how | Concrete steps |
| effort | small / medium / large |
| status | open / in_progress / done / dismissed |
| dismiss_reason | When dismissed |

### Harness application (how to put it into the user's AI)

Harness-agnostic first, with optional concrete renderings.

| Field | Notes |
|---|---|
| title | "Skill: summarize sources into notes" |
| kind | rule / skill / hook / prompt / tool (MCP) / workflow / agent |
| description | What it does and when it triggers, harness-neutral |
| rationale | Why it helps, linked to the source |
| renderings | Optional per-harness drafts: `claude_code` (CLAUDE.md snippet, SKILL.md, hook JSON), `cursor` (rule), `agents_md`, ... |
| status | proposed / accepted / applied / rejected |

Analysis is personalized: the analyzer gets the user model (goals, known harness setup, past rejections) so it doesn't produce actions/applications the user already has or has rejected.

## Quality rules for analysis

- Fewer, better items. Usually 3–10 notes, 0–5 actions, 0–3 harness applications per source.
- No item without a citation.
- Actions must be doable, not "learn more about X".
- Harness applications must name a trigger (when it fires) and an effect.
