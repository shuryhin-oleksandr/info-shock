# Personal Knowledge Library & AI Harness Advisor — Specifications

## 1. Vision & Overview

This project is a **Personal Knowledge Library and AI Workflow/Harness Advisor** ("Info-Shock" / Knowledge Engine). It ingests information sources from the user, analyzes them through the lens of the user's personal context and AI tooling setup ("AI Harness"), and extracts structured, actionable intelligence:

1. **Main Ideas (Notes)**: Distilled core concepts, mental models, and key takeaways from the source.
2. **Action Points (Actions)**: Concrete tasks and behavioral changes the user should execute based on the knowledge.
3. **Harness Applications (Harness Apps)**: Specific ways to encode and integrate this knowledge into the user's AI harness (e.g., new system rules, skills, slash commands, prompt templates, subagents, hooks, or MCP integrations).

Once stored, the user's AI assistants (such as Claude Code, Cursor, Antigravity, Windsurf, etc.) connect to this library via the **Model Context Protocol (MCP)**. Through MCP, the AI assistant can actively advise the user on what to do differently in their current workflow, apply stored knowledge to ongoing tasks, track action items, and directly generate or update AI harness configurations.

---

## 2. Core Design Decisions

| Decision Area | Chosen Approach |
| :--- | :--- |
| **Primary Interfaces** | **Dual Interface**: **Web UI** (dashboard for adding sources, browsing notes/actions/harness apps) + **MCP-First / Headless** (full ingestion, querying, and workflow advisory directly inside the AI assistant). |
| **Supported Source Types (MVP)** | Web URLs / Articles, Raw Text / Markdown notes, YouTube Videos (via transcripts), and Local Files (PDF, Markdown, Docs). |
| **Storage Architecture** | **Hybrid Local Storage**: Human-readable **Markdown files with YAML frontmatter** on disk (Git & Obsidian compatible) paired with a local **SQLite database + vector embeddings (`sqlite-vec`)** for fast relational filtering, full-text search (FTS5), and semantic search. |
| **Personalization Context** | **Configurable User & Harness Profile**: The system maintains a profile of the user's role, active projects, and AI harness setup (tools used, existing skills/rules/workflows) so extractions are tailored at ingestion time. |
| **Post-Analysis Workflow** | **Auto-Save as Active**: Extracted Notes, Action Points, and Harness Applications are immediately saved as active items upon analysis, with full ability to edit, archive, or delete them later via Web UI or MCP. |
| **Technology Stack** | **TypeScript / Node.js** monorepo or unified app (Next.js / Vite React Web UI, Hono/Fastify API, `@modelcontextprotocol/sdk` MCP Server, `better-sqlite3` + `sqlite-vec`, Vercel AI SDK for LLM structured outputs). |

---

## 3. Specification Index

- **[01 — Architecture & Storage](./01-architecture-and-storage.md)**
  System topology, tech stack details, directory layout for the Markdown vault, and synchronization between Markdown files and SQLite.
- **[02 — Data Models & Schemas](./02-data-models.md)**
  Detailed entity definitions, YAML frontmatter formats, and SQLite tables for `Source`, `Note`, `ActionPoint`, `HarnessApplication`, and `UserProfile`.
- **[03 — Ingestion & Analysis Pipeline](./03-ingestion-and-analysis.md)**
  Content extractors (URL, YouTube, PDF/Doc, Raw Text), chunking strategy, LLM analysis prompts, structured output schemas, and embedding generation.
- **[04 — MCP Server Specification](./04-mcp-server.md)**
  MCP tools, resources, and prompts for the **Context/Workflow Advisor**, **Harness Generator**, **Knowledge Search**, **Action Tracker**, and **Headless Ingestion**.
- **[05 — Web UI Specification](./05-web-ui.md)**
  Screens, user flows, and component architecture for the browser dashboard.
- **[06 — Implementation Roadmap](./06-roadmap.md)**
  Phased milestones from core foundation to full MCP workflow advisor and Web UI.
