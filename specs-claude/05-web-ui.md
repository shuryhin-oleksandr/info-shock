# 05 — Web UI

Local web app (localhost). No login in v1.

## Screens

### Inbox / Add source
- Paste URL, paste text, upload file (drag and drop).
- Optional "why I'm saving this".
- List of recent sources with pipeline status, errors and retry.

### Source detail
- Metadata, summary, extracted text (collapsible).
- Its notes, actions and harness applications, each with its citation highlighted in the text.
- Edit / delete items; re-run analysis.

### Library
- Tabs: Notes, Actions, Harness applications.
- Full-text + semantic search, filters (tag, source, status, importance).
- Actions: status changes (open / in progress / done / dismissed with reason).
- Harness applications: view renderings per harness, copy to clipboard, mark accepted/applied/rejected.

### Advice log
- Every suggestion the advisor gave: when, in which session, context, evidence.
- Decision history (explicit/inferred) with reasons; the user can correct a decision.

### Sessions
- List of reported sessions: client, project, goal, outcome.
- Detail: summary, suggestions shown, decisions, inferred outcomes, transcript (if present).

### Profile
- User model: goals, preferences, harness, patterns. Editable.
- Change log of model-proposed updates, with revert.

### Insights
- Metrics from 04-learning-loop.md.

### Settings
- LLM provider + model per task (analysis, advising, relevance check, embeddings, transcription), API keys.
- Advice level: quiet / normal / active.
- MCP connection info: config snippets for common clients (Claude Code, Cursor, Claude Desktop).
- Claude Code adapter install instructions.
- Data: export (JSON/Markdown), delete everything.
