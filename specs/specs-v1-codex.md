# Specs v1 — Codex

## Goal

Build one small, working loop: save knowledge from pasted text and consult it through a separate MCP server during an AI task.

## How it works

1. Start the local app and connect its MCP server to your existing AI assistant.
2. Paste source text into a simple page and click **Analyze**.
3. The app uses one configured model provider to extract **notes**, **actions**, and **harness improvement ideas**.
4. Save the source and extracted insights locally. Display them on the page, with each insight linked to its source. Actions and harness ideas may be empty when the source offers none.
5. During a task, ask your AI: **“Check my library for relevant advice.”**
6. Your AI sends the current task context to one MCP tool.
7. The server searches the saved insights and returns relevant advice with source evidence, or no advice when nothing fits.
8. Your AI presents the advice. You decide what to apply.

## First-version scope

- One local user, one page, and local persistent storage.
- Pasted text as the only source input.
- One configured model provider for analysis and advice.
- A separate MCP server exposing one tool for task-specific library advice.
- Consultation happens when the user requests it.
- Harness ideas remain saved suggestions. The app and server do not create or modify skills, commands, or AI configuration.

## Deferred

Accounts, hosting, URL and file extraction, automatic check-ins, feedback tracking, and automatic harness editing.

## Done when

- A user can paste and analyze text, then see its saved notes, actions, and harness ideas.
- Saved knowledge remains available after restarting the app.
- In a separate AI conversation, the user can request advice and receive a relevant suggestion grounded in the saved source.
- An unrelated task produces no advice.
- Failed analysis displays an error without reporting a successful save.

Example: save an article about testing. Later, while fixing a bug, ask the AI to consult the library. It suggests a regression test and identifies the saved article as its evidence.
