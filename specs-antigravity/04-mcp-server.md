# 04 — MCP Server Specification

## 1. Overview & Role of the MCP Server

The MCP Server (`knowledge-library-mcp`) bridges the user's stored knowledge base with their active AI coding/workflow assistant (Antigravity, Claude Code, Cursor, etc.).

Instead of the knowledge sitting passively in notes, the AI assistant uses MCP to:
1. **Advise on Workflow (`advise_on_workflow`)**: Given what the user is currently working on or how they are approaching a problem, retrieve relevant Notes, Action Points, and Harness Applications and tell the user *what to do differently* based on their knowledge library.
2. **Generate/Apply Harness Upgrades (`get_harness_recommendations`, `mark_harness_applied`)**: Pull draft rules, skills, hooks, or prompts from saved Harness Applications and directly write/update them in the user's workspace or global agent configuration.
3. **Search & Explore Knowledge (`search_knowledge`, `get_entity_details`)**: Semantic + keyword search across all Sources, Notes, Action Points, and Harness Applications.
4. **Track & Execute Action Points (`list_action_points`, `update_action_status`)**: Surface pending Action Points relevant to the current task and mark them `in_progress` or `completed`.
5. **Headless Ingestion (`ingest_source`)**: Add a URL, YouTube video, local file, or raw thought directly from the AI chat without opening the browser.

---

## 2. MCP Tools Reference

### 2.1 Workflow & Context Advisor

#### `advise_on_workflow`
The signature tool of the product. The AI assistant calls this tool with the user's current task, plan, code diff, or workflow description to get actionable feedback grounded in the user's personal knowledge library.

- **Input Schema**:
  ```json
  {
    "current_task_or_context": "string (What the user/agent is currently trying to build, debug, or plan)",
    "current_approach": "string (Optional: How the user or agent is currently doing or planning to do it)",
    "max_recommendations": "number (default: 5)"
  }
  ```
- **Behavior**:
  1. Runs hybrid semantic + FTS5 search against `notes` (matching `when_to_apply` & `core_idea`), `action_points` (matching `trigger_context`), and `harness_applications` (matching `trigger_context`).
  2. Returns the top matched items along with a structured synthesis instruction (or LLM-synthesized critique) highlighting:
     - **What to do differently** in the current workflow based on saved Main Ideas.
     - **Pending Action Points** triggered by this task.
     - **Harness Applications** that could automate or enforce this improvement right now.

---

### 2.2 Harness Generator & Integration Tools

#### `list_harness_applications`
Lists saved AI Harness Applications filtered by status, target tool, or component type.

- **Input Schema**:
  ```json
  {
    "status": "'proposed' | 'applied' | 'archived' | 'all' (default: 'proposed')",
    "target_tool": "'antigravity' | 'claude_code' | 'cursor' | 'generic' | 'all' (optional)",
    "target_component": "'skill' | 'rule' | 'slash_command' | 'mcp' | 'hook' | 'subagent' | 'prompt' (optional)",
    "context_query": "string (Optional semantic filter for current task)"
  }
  ```
- **Output**: List of matching `HarnessApplication` items including `problem_solved`, `proposed_change`, `trigger_context`, and `draft_snippet`.

#### `update_harness_status`
Updates the status of a Harness Application (e.g., marking it as `applied` after the AI assistant creates the corresponding `SKILL.md` or `.agents/rules/` file in the user's repo) or updates its `draft_snippet`.

- **Input Schema**:
  ```json
  {
    "id": "string (e.g., 'hrn_01j9x8d')",
    "status": "'proposed' | 'applied' | 'archived'",
    "applied_location": "string (Optional path where the rule/skill was written, e.g., '.agents/rules/poka-yoke.md')"
  }
  ```

---

### 2.3 Knowledge Search & Retrieval Tools

#### `search_knowledge`
Performs hybrid (semantic vector + FTS5 keyword) search across the entire knowledge base.

- **Input Schema**:
  ```json
  {
    "query": "string (Natural language question or keywords)",
    "entity_types": "Array<'note' | 'action' | 'harness' | 'source'> (default: all)",
    "tags": "Array<string> (Optional tag filter)",
    "limit": "number (default: 10)"
  }
  ```

#### `get_knowledge_item`
Fetches the full Markdown content, metadata, and linked entities (parent Source, sibling Notes/Actions/Harness items) for one or more entity IDs.

- **Input Schema**:
  ```json
  {
    "ids": "Array<string> (e.g., ['note_01j9x8a', 'src_01j9x800'])"
  }
  ```

---

### 2.4 Action Point Tracker Tools

#### `list_action_points`
Retrieves Action Points filtered by status, priority, or relevance to a task context.

- **Input Schema**:
  ```json
  {
    "status": "'todo' | 'in_progress' | 'completed' | 'archived' (default: 'todo')",
    "priority": "'high' | 'medium' | 'low' (optional)",
    "context_query": "string (Optional: filter actions whose trigger_context matches current work)"
  }
  ```

#### `update_action_point`
Updates the status (`todo`, `in_progress`, `completed`, `archived`) or edits an Action Point.

- **Input Schema**:
  ```json
  {
    "id": "string (e.g., 'act_01j9x8c')",
    "status": "'todo' | 'in_progress' | 'completed' | 'archived'",
    "resolution_note": "string (Optional note on how the action was completed)"
  }
  ```

---

### 2.5 Headless Ingestion & Management Tools

#### `ingest_source`
Allows the user to feed a URL, YouTube link, local file path, or raw text/insight directly through their AI assistant ("Save this article to my knowledge library and tell me how it applies to our project").

- **Input Schema**:
  ```json
  {
    "input": "string (URL, YouTube link, absolute file path, or raw text)",
    "source_type": "'auto' | 'url' | 'youtube' | 'file' | 'text' (default: 'auto')",
    "title": "string (Optional override title)",
    "focus_prompt": "string (Optional instructions on what to focus on during extraction)"
  }
  ```
- **Output**: Summary of the ingested `Source` along with all auto-saved `Notes`, `ActionPoints`, and `HarnessApplications`.

#### `update_user_profile`
Reads or updates `vault/profile.md` so the AI assistant can keep the user's current project context and harness setup up to date.

---

## 3. MCP Prompts & Resources

### MCP Prompts (Slash Commands in MCP Clients)
- `review-my-workflow`: Prompt template that instructs the host AI to inspect the current workspace/conversation, call `advise_on_workflow`, and present a structured critique of what the user should do differently based on their stored knowledge.
- `upgrade-my-harness`: Prompt template that inspects the current repository's agent config (`.agents/`, `CLAUDE.md`, `.cursorrules`, etc.), calls `list_harness_applications`, and proposes concrete file creations/edits to integrate pending Harness Applications.

### MCP Resources
- `knowledge://profile` — Returns the current `vault/profile.md`.
- `knowledge://actions/todo` — Live feed of high-priority pending Action Points.
- `knowledge://harness/proposed` — Live feed of unapplied Harness Applications.
