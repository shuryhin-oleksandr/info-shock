# Acceptance criteria

Status: draft scenarios derived from the product decisions. These describe required behavior without selecting an implementation.

## Source ingestion and knowledge

- A user can submit a web URL, pasted text, a PDF, or a YouTube URL through the web UI or their AI conversation.
- A successfully processed source produces notes, actions, and AI configuration applications that are immediately available without review.
- An unreadable source produces a clear error report. OCR and transcription of videos without available transcripts are outside the first version.
- Submitting an existing source reuses its entry and refreshes extracted knowledge while preserving user edits.
- Users can browse and edit extracted knowledge through either interface. Future suggestions use user edits in preference to the original extraction.
- Each user's library is private. One user's sources and preferences do not become available to another user.
- Deleting a source removes it and its extracted knowledge from active use and future suggestions. Previously applied changes remain in place.
- A source that contributed to a suggestion with a recorded result is soft deleted to preserve history and statistics. Approve, decline, and decline forever each count as results. A source with no recorded results is deleted normally, including one with only unanswered suggestions.

## Consultation and suggestions

- A consultation receives the latest user message and a short summary of the current task.
- Consultation frequency is configurable, including support for every N user messages. The complete setting list and counting rules remain open in product.md.
- A consultation returns at most three suggestions, each requiring a strong match to the current work. If no strong matches exist, it returns no suggestions.
- Each suggestion explains why it applies and cites its source.
- When relevant sources conflict, the user sees the alternatives and a recommended option with an explanation.
- If consultation is slow or unavailable, the user's AI continues without library advice and briefly indicates that the check could not complete.

## Approval and application

- Before approval, a suggestion shows the exact proposed change; configuration edits include a before/after diff.
- Suggestions can affect either the current task or persistent AI configuration.
- Each suggestion offers approve, decline, and decline forever.
- After approval, the user's existing AI applies the proposed change using its own tools and permissions.
- A temporarily declined suggestion can return in a new conversation or after a substantial context change, provided it still strongly matches.
- A permanently declined recommendation stays suppressed for that user even when another source recommends the same thing. Unrelated recommendations from those sources remain eligible.
- Users can manage permanent declines and undo suppression through either interface.

## History and reversal

- Suggestion history shows the proposals, their sources, and the user's approval or decline decisions.
- The product does not provide an undo mechanism for applied changes in the first version. Reversal is handled by the user's existing AI.
- Approval is recorded separately from application status. The user's AI reports successful application or failure, and history records the reported outcome. Approval alone is not proof of successful application.
- Soft-deleted sources preserve associated suggestion history and statistics but cannot inform future suggestions. Retained source fields remain open.
- Statistics show suggestions shown, approvals, declines, permanent declines, successful applications, and failed applications, both as totals and broken down by source.
- An unanswered suggestion creates no result record but still counts toward suggestions shown.

## Integration scope

- Users sign in to a hosted service and connect their AI without deploying or managing their own server.
- Onboarding provides simple plugin-style connection and consultation setup, without complex installation, a separately managed local helper, manual configuration edits, or manually added instructions. Client-specific setup mechanisms remain to be investigated.
- Suggestions appear in the user's existing AI conversation.
- The product targets web chat, desktop AI applications, and CLI coding agents across providers, including Claude Code, Codex, and Gemini across applicable interfaces. Exact supported clients and rollout order remain open.
- Client availability, setup mechanisms, regular consultation, and ability to apply approved changes must be verified before client-specific acceptance checks can be finalized. Universal compatibility is a product goal, not yet a validated release criterion.
- A client without automatic consultation may be supported using user-triggered consultation. Users must be told whether their connected client supports automatic or user-triggered checks.
