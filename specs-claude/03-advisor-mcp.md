# 03 — Advisor MCP

The MCP server is the product's main surface. It connects to the user's own AI. The user's AI
stays in charge: it calls the advisor, weighs what it gets back, and decides with the user.

## Transports

- **stdio**, for clients that launch the server as a process.
- **Streamable HTTP on localhost**, for clients that connect to a running app (and for adapters/hooks).

Both are backed by the same local app and database.

## Advice format

Every advising tool returns suggestions in one shape, so the user's AI can evaluate them:

```jsonc
{
  "suggestions": [
    {
      "id": "sug_123",
      "type": "approach | critique | harness | action_reminder | knowledge",
      "title": "Write the test before fixing the bug",
      "advice": "What to do, concretely",
      "reasoning": "Why, applied to the user's current situation",
      "evidence": [{ "item": "note_42", "source": "src_7", "quote": "..." }],
      "confidence": "low | medium | high",
      "previously": "User rejected a similar suggestion on 2026-10-02: 'too slow for prototypes'"
    }
  ],
  "nothing_to_add": false
}
```

Rules:
- An empty answer (`nothing_to_add: true`) is a valid and frequent result.
- `previously` is filled when there's related history, so the user's AI knows the context.
- Every returned suggestion is stored and tracked (see 04-learning-loop.md).
- The response text asks the user's AI to evaluate before applying, and to report the outcome with `record_feedback`.

## Tools

### Advising

| Tool | When the AI calls it | Input | Output |
|---|---|---|---|
| `get_briefing` | Start of a task/session | goal, project/context, harness name | Relevant suggestions + open actions related to the goal |
| `advise` | Before a key decision or plan | goal, current plan/approach, constraints | Suggestions: better approaches, critique of the plan |
| `review` | After a chunk of work | what was done, result, problems hit | Critique + fixes + suggestions |
| `suggest_harness_refinements` | When asked, or when the advisor flagged a pattern | harness name, current config excerpts (optional) | Harness suggestions with renderings |

### Feedback and sessions

| Tool | Input | Effect |
|---|---|---|
| `record_feedback` | suggestion id, decision (accepted / rejected / deferred / applied), reason, usefulness 1–5 | Stored; updates the user model |
| `report_session` | session id, goal, summary of what happened, outcome, suggestions used, optional transcript | Stored; triggers inference (see 04) |

### Library

| Tool | Input | Output |
|---|---|---|
| `search_library` | query, kinds (note/action/harness), limit | Matching items with citations |
| `get_item` | id | Full item |
| `add_source` | url, or text, or file (path or base64), optional note on why it was saved | source id + status |
| `get_source_status` | source id | Pipeline status, created items |
| `list_actions` | status filter, topic | Actions |
| `update_action` | id, status, note | Updated action |

### Resources (read-only)

- `library://profile` — the user model summary (goals, preferences, known harness).
- `library://actions/open` — open actions.

## Check-in protocol (server instructions)

MCP server instructions (sent to the client on connect) tell any AI how to work with the advisor.
Draft:

> You are connected to the user's personal knowledge advisor.
> - At the start of a task, call `get_briefing` with the goal.
> - Before committing to a plan or a non-trivial decision, call `advise`.
> - After finishing a meaningful piece of work, call `review`.
> - Evaluate each suggestion yourself. Apply the ones that hold up, tell the user about important ones, and push back on weak ones.
> - Report each decision with `record_feedback`, including why.
> - At the end of the session, call `report_session`.
> - When the user shares something worth keeping, offer to `add_source`.

Instructions alone are best-effort: clients may ignore them. Adapters make check-ins reliable.

## Adapters

### Claude Code adapter (v1)

Installed as a small package or plugin that registers hooks calling the local app over HTTP:

| Hook | Behavior |
|---|---|
| SessionStart | Fetch a briefing for the project; inject it as context |
| UserPromptSubmit | Fast relevance check on the prompt; inject advice **only** above a relevance threshold |
| Stop / SessionEnd | Send the session transcript (redacted) for `report_session` |

### Noise and cost control (for continuous mode)

- Two-stage check: cheap retrieval + relevance scoring first; the full reasoning model only when relevance is high.
- Cooldown: don't inject advice on consecutive prompts unless it's high confidence.
- Don't repeat a suggestion already shown in the session or rejected before (unless there's new evidence).
- User-configurable level: `quiet` (briefing + on request) / `normal` / `active`.

### Other harnesses

v1 relies on server instructions. Adapters for Cursor, Codex etc. come later, if their hook systems allow it.
