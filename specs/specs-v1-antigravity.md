# Personal Knowledge Library & AI Harness Advisor — v1 Minimal Spec

## 1. Overview & Philosophy

Instead of building a heavy standalone web application with its own LLM billing, database, and UI, **v1** uses a **Host-AI-Driven + Lightweight MCP Storage** architecture:

- **Your AI Assistant (Claude Code, Antigravity, Cursor, etc.) is the Brain & UI**: It fetches/reads sources, analyzes them using its own model, advises you during daily tasks, and directly writes/updates its own harness files (skills, rules, slash commands, hooks) when you approve.
- **The MCP Server (`info-shock-mcp`) is the Shared Memory & Protocol**: A ~150-line zero-API-key local MCP server backed by plain Markdown files (`vault/`). It stores your extracted **Notes**, **Actions**, and **Harness Applications**, searches them when you start a task, and instructs any connected AI when to check in.

---

## 2. Storage Layout (`vault/`)

All data lives in human-readable Markdown files on disk (editable in VS Code, Obsidian, or Git):

```text
vault/
├── profile.md       # Short description of your role, stack, and AI harness setup
├── notes.md         # Extracted Main Ideas (with source citation & when-to-apply trigger)
├── actions.md       # Extracted Action Points (- [ ] todo / - [x] done)
└── harness.md       # Extracted AI Harness Ideas (proposed / applied rules, skills, commands)
```

---

## 3. Step-by-Step End-to-End Flow

### Step 1: Ingest & Analyze a Source
1. **User prompts their AI**: *"Analyze this source for my library: `<URL or text>`"*
2. **Host AI** fetches/reads the source content and reads `vault/profile.md` (via MCP).
3. **Host AI** extracts three structured categories:
   - **Notes**: Main ideas + *when to apply* them.
   - **Actions**: Concrete things the user should do + *trigger context*.
   - **Harness Applications**: Rules, skills, slash commands, or hooks to improve the AI setup + a draft snippet.
4. **Host AI** calls the MCP tool `save_knowledge(...)` to append the extracted items to `notes.md`, `actions.md`, and `harness.md`.

### Step 2: Upgrade the AI Harness
1. Right after saving (or whenever relevant), the **Host AI** presents the extracted **Harness Applications**:
   > *"From this source, I can add a rule to `.agents/rules/` or create a new `/verify` skill. Want me to apply #1?"*
2. **User approves**: *"Yes, apply #1."*
3. **Host AI** uses its own file-editing tools to create/update the skill, rule, or command in your workspace/global config, and calls the MCP tool `update_item_status` to mark that harness item as `applied`.

### Step 3: Daily Workflow Advice (What to Do Differently)
1. **User starts a normal task** in any repo: *"Let's refactor the auth middleware."*
2. Guided by the MCP Server Instructions, the **Host AI** calls `check_knowledge(task="refactor auth middleware")` before finalizing its plan.
3. **MCP Server** searches `notes.md`, `actions.md`, and `harness.md` (keyword/tag matching across titles, body, and `when_to_apply` triggers) and returns matching items.
4. **Host AI advises the user**:
   > *"Before we start: based on the article you saved earlier, we should isolate token validation first (Action #act-3) instead of mixing it into the route handler. Proceed this way?"*
5. **User agrees** $\rightarrow$ Host AI executes the improved workflow and calls `update_item_status` to mark the action `done`.

---

## 4. MCP Server Specification (`info-shock-mcp`)

### 4.1 Server Instructions (Auto-Check-In Protocol)
Sent automatically to the AI client upon MCP connection:
> You are connected to the user's Personal Knowledge Library (`info-shock-mcp`).
> 1. **When the user shares a link, article, or text to save/analyze**: Read it, extract main ideas (`notes`), user action points (`actions`), and AI workflow/config upgrades (`harness`), then call `save_knowledge` and offer to apply any useful harness rules/skills immediately.
> 2. **Before starting a new task or finalizing an implementation plan**: Call `check_knowledge` with the current task description to see if the user's saved notes or pending actions suggest a better approach. If a strong match is found, briefly tell the user what to do differently and cite the source.
> 3. **When an action is completed or a harness rule/skill is written**: Call `update_item_status` to mark it `done` or `applied`.

### 4.2 MCP Tools (Only 3 Tools)

#### 1. `save_knowledge`
Saves extracted intelligence from a source into `vault/{notes,actions,harness}.md`.
- **Inputs**:
  - `source_title`: string
  - `source_uri`: string (optional URL or file path)
  - `notes`: `Array<{ title, core_idea, when_to_apply, tags }>`
  - `actions`: `Array<{ title, what_to_do, trigger_context }>`
  - `harness_apps`: `Array<{ title, target_type ('skill' | 'rule' | 'command' | 'hook' | 'prompt'), proposed_change, draft_snippet }>`
- **Output**: Assigned IDs (`note-1`, `act-1`, `hrn-1`) and confirmation.

#### 2. `check_knowledge`
Retrieves relevant Notes, pending Actions, and unapplied Harness Ideas for the current task.
- **Inputs**:
  - `query`: string (keywords or task description, e.g. `"auth refactor testing"`)
  - `include_all_open`: boolean (optional — if true or if the vault is small, returns all active notes/open actions so the host AI can semantically filter them in-context)
- **Output**: Matching `notes`, `todo` actions, `proposed` harness items, plus `profile.md`.

#### 3. `update_item_status`
Marks an Action Point or Harness Application as completed, applied, or dismissed.
- **Inputs**:
  - `id`: string (e.g. `act-1` or `hrn-2`)
  - `status`: `'done' | 'applied' | 'dismissed'`
  - `note`: string (optional, e.g. path where the skill/rule was written)
- **Output**: Updated item status.

---

## 5. Technical Stack & Implementation Scope

- **Runtime**: Node.js + TypeScript (single entry file `src/index.ts`).
- **Dependencies**: `@modelcontextprotocol/sdk` + `zod` (zero external DBs, zero LLM SDKs required).
- **Transport**: `stdio` (works out of the box with Claude Code, Antigravity, Cursor, Codex).
- **Search Strategy for v1**: Keyword scoring over Markdown sections + `include_all_open` fallback (since the first ~50–100 sources easily fit inside modern AI context windows, allowing the host AI to do semantic matching for free).
