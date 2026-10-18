# Specs v1 — smallest working version

A thin version of the whole loop: **save a source → extract knowledge → the user's AI asks for advice → get advice with a source link.**
Every part exists, in its simplest form.

## Scope

| Part | v1 version |
|---|---|
| Inputs | Web article URL and pasted text only |
| Analysis | One LLM call per source, returning notes, actions and harness applications as JSON |
| Storage | One local SQLite file, keyword search (FTS5). No vector search |
| Interface | Separate MCP server, local, stdio. No web UI |
| Trigger | MCP server instructions tell the AI when to call `advise`. No hooks |
| Users | One user, on their own machine |

## MCP tools

| Tool | What it does |
|---|---|
| `add_source(url \| text)` | Fetch, analyze, save; return what was extracted |
| `search_library(query)` | Find saved notes, actions and harness applications |
| `advise(task, approach?)` | Find matching items, one LLM call → up to 3 suggestions, each with a reason, a source link and the exact change. Can return "nothing to add" |

## How it works

1. **Setup.** The user adds the MCP server to Claude Code once. Its instructions tell Claude to call `advise` at the start of a task.
2. **Add a source.** The user says "save this article: <url>". Claude calls `add_source`.
3. **Analysis.** The server fetches the text and makes one LLM call that returns notes, actions and harness applications.
4. **Storage.** The server saves them in its SQLite database. Nothing changes in the user's Claude setup.
5. **Work later.** The user starts a task ("set up tests"). Claude calls `advise` with the task.
6. **Matching.** The server searches the database for related items. If nothing matches, it returns "nothing to add".
7. **Advice.** If items match, one LLM call writes up to 3 suggestions: reason, source link, exact change.
8. **Proposal.** Claude shows the suggestions and asks whether to apply them.
9. **Apply.** If the user says yes, Claude makes the change itself: edits the task, or writes to `CLAUDE.md`, a skill or a command.
10. **Effect.** From the next session on, Claude follows the new rule or skill automatically.

## Who writes to the harness

The MCP server never touches the user's files. It only returns text. The user's AI applies an approved change with its own tools, under its own permission prompts.

| Harness part | File Claude Code edits |
|---|---|
| Rule | `CLAUDE.md` (global or project) |
| Skill | `.claude/skills/<name>/SKILL.md` |
| Slash command | `.claude/commands/<name>.md` |
| Hook | `.claude/settings.json` |

Why:
- Works with any AI. Codex or Cursor apply the same suggestion to their own files (`AGENTS.md`, Cursor rules).
- The user stays in control and sees the diff.
- The server needs no file access, so a malicious article can't make it write anything.

Harness applications from analysis stay in the library as proposals. They reach the user's setup only when relevant to a task and approved.

## Known weak spot

Step 5 depends on the AI following the server instructions, and it won't always call `advise`. In v1 the user can also say "check my library". Later, a `UserPromptSubmit` hook makes it automatic.

## Out of scope for v1

Web UI, PDF / YouTube / podcasts, vector search, feedback and learning loop, "decline forever", Claude Code hooks, user profile, hosting, accounts.

## Done when

1. The user adds 2–3 real articles from Claude Code.
2. In a later, unrelated chat, `advise` returns a relevant suggestion that names its source.
3. When nothing in the library matches, `advise` returns nothing.

## Next steps, in order

`record_feedback` (approve / decline) → simple web UI → PDF / YouTube → vector search → hooks for automatic check-ins.
