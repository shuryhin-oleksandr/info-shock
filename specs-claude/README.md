# Specs

A personal knowledge library that turns what you read into an **advisor** for your own AI.

You feed it sources (articles, PDFs, videos, notes). It extracts ideas, actions and harness
improvements. Then, through MCP, it sits next to whatever AI assistant you already use and
advises it: proposes better ways to reach your goals, criticizes the current approach, and
suggests harness refinements (rules, skills, hooks). It remembers what you accepted or
rejected, and why, and gets better at advising you.

## Files

| File | What it covers |
|---|---|
| [01-product.md](01-product.md) | Vision, principles, user, scope, non-goals |
| [02-ingestion.md](02-ingestion.md) | Sources, analysis pipeline, notes / actions / harness applications |
| [03-advisor-mcp.md](03-advisor-mcp.md) | MCP server: tools, instructions, check-in protocol, adapters |
| [04-learning-loop.md](04-learning-loop.md) | Sessions, suggestion tracking, feedback, user model |
| [05-web-ui.md](05-web-ui.md) | Web UI screens |
| [06-architecture.md](06-architecture.md) | Stack, components, data model, LLM providers, privacy |
| [07-open-questions.md](07-open-questions.md) | Undecided items |

## Glossary

- **Source** — anything the user gives the library: URL, pasted text, PDF, YouTube video, podcast.
- **Note** — a main idea extracted from a source.
- **Action** — something the user is supposed to *do* with a source's information.
- **Harness application** — a way to build the knowledge into the user's AI harness (a rule, skill, hook, prompt, tool, workflow).
- **Harness** — the user's AI setup: the assistant plus its config (CLAUDE.md / AGENTS.md / Cursor rules, skills, hooks, MCP servers).
- **User's AI** — the assistant the user chooses (Claude Code, Cursor, Codex, ChatGPT, ...). Not part of this product.
- **Advisor** — this product's MCP server plus its own reasoning model.
- **Suggestion** — one piece of advice the advisor gave, with an id, tracked until accepted, rejected or applied.
- **Session** — one working session of the user with their AI, as reported to the advisor.

## Key decisions (from spec discussion, 2026-10-18)

1. Single user, runs locally. Each user runs their own instance.
2. The user's AI is their own choice. The product is **only an advisor**, plugged in via MCP. No vendor lock-in.
3. Harness-agnostic. Suggestions are described generically, with optional renderings per harness.
4. The advisor reasons with its **own model**. Provider set in config (Anthropic, OpenAI, Gemini, Ollama, ...).
5. Advice always comes with its evidence (library items) and reasoning, so the user's AI can check it before applying it.
6. "Continuous" advice = MCP server instructions that tell any AI when to check in, plus optional adapters (Claude Code hooks first) that enforce it.
7. Sources can be added from the web UI **and** through MCP by the user's AI.
8. Feedback: the AI reports outcomes explicitly, and the advisor infers from later sessions whether a suggestion was applied. Every decision is remembered with its reason.
9. Stack: TypeScript, local.
10. v1 = everything, including session tracking and the learning loop.
