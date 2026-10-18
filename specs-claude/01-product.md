# 01 — Product

## Problem

People consume a lot of useful material (articles, talks, docs) and then do nothing with it.
The knowledge never reaches daily work, and especially never reaches the AI assistant
they work with all day.

## Vision

An **augmented human** layer. The user works with their own AI as usual. In the background,
this product watches the work through MCP and, based on the user's library, says things like:

- "There's a better way to do what you're trying to do. Here's how, from source X."
- "This approach has a problem: ... Recommended fix: ..."
- "You keep doing Y manually. A skill/hook for it would look like this."
- "You saved an article on Z last week, and it applies here."

## User

One person who:
- works daily with an AI assistant (coding or otherwise),
- regularly reads or watches material they want to apply,
- is willing to run a local app and connect an MCP server.

## Principles

1. **Advisor, not driver.** The product never acts on the user's machine. It advises. The user's AI and the user decide.
2. **Show your evidence.** Every suggestion points to the library items it's based on and explains why.
3. **Bring your own AI.** Works with any MCP-capable assistant. Harness-specific features are optional extras.
4. **Bring your own model.** The advisor's model provider is configurable.
5. **Learn the user.** Remember what was accepted, rejected and applied, and why. Don't repeat rejected advice without a new reason.
6. **Low noise.** Speak up when it matters. Silence is a valid answer.
7. **Local and private.** Data stays on the user's machine, except calls to the configured model provider.

## v1 scope

- Ingest: URL articles, pasted text/Markdown, PDF/documents, YouTube, podcasts. From the web UI and from MCP.
- Analysis: extract notes, actions, harness applications per source; link to existing library items.
- Library: browse, search, edit, delete; action status tracking.
- Advisor MCP: briefing, advise/critique, harness refinement, search, add source, feedback, session reporting.
- Check-in protocol in the MCP server instructions + a Claude Code adapter (hooks).
- Learning loop: suggestion tracking, explicit feedback, inference from sessions, user model.
- Web UI for all of the above.
- Configurable LLM provider.

## Non-goals (v1)

- Multi-user, accounts, billing, hosting.
- Mobile app or chat bots.
- The product editing the user's harness files itself (it proposes; the user's AI applies).
- Its own chat UI for talking to the advisor (the user's AI is the chat).
- Adapters for harnesses other than Claude Code (the protocol still works for them through instructions).

## Success signals

- Share of suggestions accepted, and share actually applied.
- Actions moved to "done".
- The user keeps adding sources (the library is worth feeding).
- Rejected advice doesn't come back without new grounds.
