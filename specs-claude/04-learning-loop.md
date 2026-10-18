# 04 — Learning loop

The advisor gets better by remembering how its advice landed and how the user works.

## Suggestion lifecycle

```
shown → accepted → applied
      ↘ rejected
      ↘ deferred → (shown again later)
      ↘ ignored (no feedback, session ended)
```

Each suggestion stores: id, type, content, evidence, the context it was given in (goal, session),
every decision with reason and timestamp, and how the decision was learned (`explicit` from the AI or the UI, or `inferred`).

## Sources of feedback

1. **Explicit**: the user's AI calls `record_feedback` (decision + why). The user can also change a decision in the web UI.
2. **Inferred**: after `report_session`, the advisor compares the session against open/accepted suggestions and decides:
   - applied (evidence in the session that it was done),
   - ignored,
   - contradicted (the user did the opposite).
   Inferred decisions are marked as such, with the session excerpt as evidence, and never override an explicit one.

## Sessions

A session record holds: client/harness, project, goal, summary, outcome, suggestions shown,
decisions made, and optionally the transcript (Claude Code adapter).

Sessions are used to:
- infer suggestion outcomes,
- detect recurring patterns ("user fixes the same lint issue by hand every session") → candidate harness suggestions,
- update the user model.

## User model

A structured profile, kept up to date by the advisor and editable by the user:

| Part | Examples |
|---|---|
| Goals | "Ship the MVP by December", "Get better at testing" |
| Preferences | "Prefers small PRs", "Dislikes heavy frameworks" |
| Harness | Which AI tools, known rules/skills/hooks already in place |
| Patterns | Recurring mistakes, recurring manual work |
| Advice history | What kinds of advice get accepted/rejected, and common reasons |

Updates are proposed by the model after sessions/feedback and logged (what changed, why, from which session),
so the user can review and revert them in the UI.

## How learning changes advice

- Rejected suggestions are not repeated unless new evidence appears; then `previously` explains the history.
- Suggestion types and topics with low acceptance get a higher bar (higher relevance threshold).
- Goals boost related library items in retrieval.
- Known harness setup removes suggestions the user already has.

## Metrics (shown in UI)

- Acceptance rate and applied rate, overall and per type.
- Average usefulness score.
- Most useful sources (by applied suggestions).
