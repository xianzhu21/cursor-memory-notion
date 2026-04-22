# VAN Command - Initialization & Entry Point

This command initializes the Memory Bank system, performs platform detection, determines task complexity, and routes to appropriate workflows.

To **switch** the config primary task to another **existing** Task ID and **load** that task’s progress **without** this full init (no complexity step, no whole-page Progress rewrite), use **`/change-task`** instead.

**MANDATORY: When /van is invoked, ALWAYS execute the full workflow** (Steps 1–6). The "full workflow" means **only** these 6 initialization steps—NOT implementation of the user's task. Do NOT skip steps. Do NOT substitute with other actions (e.g. only updating rules or files) even if the user message includes additional text. If the user message contains text that could be a task description (e.g. "Add X", "Clarify Y", "明确 Z"), treat it as the `/van [description]` and run Task Creation/Validation accordingly.

**CRITICAL for Level 2-4:** VAN **never** implements code, rules, or docs. For Level 2-4 tasks, VAN **stops** after Step 6 and hands off to `/plan`. Implementation is done in `/build`, not in VAN.

## Memory Bank Integration (Notion)

**CRITICAL:** This project uses **Notion** as Memory Bank. Use **`*PageUrl`** values (or **`projectPageUrl` / `taskPageUrl`**) from `.cursor/notion-memory-bank.json`—full **`https://`…** Notion page URLs for `notion-fetch` / updates (agents derive internal create-parent fields from those URLs in-session only; they are not stored as bare ids in JSON):
- **tasks** - Task page body: resolve the row whose Notion **`Task ID`** equals config `taskId` via **`Core/notion-task-id-resolution.mdc`** ( **`notion-fetch` Project** → read **`Tasks`** relation URLs → **parallel `notion-fetch`** Task pages until **`properties` → `Task ID`** matches — **not** Tasks–database semantic `notion-search` on a bare numeric id).
- **issueId** / **issueUrl** - External tracker key and browse URL; **not** the Notion **Task ID**. Usually read from the Task’s **Issue ID** **text** property (often `[KEY](https://...)` in one field); `issueUrl` is parsed from that link unless the database has a separate URL column. Synced when notion-verification runs — same Task fetch as `taskName`.
- **activeContext** - activeContext page (`activeContextPageUrl`). May contain **multiple** parallel tasks; see `Core/memory-bank-paths.mdc` **Active Context: multiple in-flight tasks**. `/van` updates **only** the section for config **`taskId`**.
- **progress** - progress page (`progressPageUrl`)
- **projectBrief** - Project page body: resolve Project by **`projectId`** per **`Core/notion-verification.mdc`** step 4 (**Project ID** is primary, like **Task ID**; **`projectName`** is optional disambiguation only—always verify **Project ID**; avoid workspace-only search on a bare numeric `projectId` first).

After Project and Task URLs are resolved, use **`notion-fetch`** / **`notion-update-page`** / **`notion-create-pages`** as required by **notion-verification** and this command’s Step 6.

## Progressive Rule Loading

This command loads rules progressively to optimize context usage:

### Step 1: Load Core Rules (Always Required)
```
Load: .cursor/rules/isolation_rules/main.mdc
Load: .cursor/rules/isolation_rules/Core/memory-bank-paths.mdc
Load: .cursor/rules/isolation_rules/Core/notion-retry.mdc
Load: .cursor/rules/isolation_rules/Core/platform-awareness.mdc
Load: .cursor/rules/isolation_rules/Core/notion-verification.mdc
Load: .cursor/rules/isolation_rules/Core/notion-task-id-resolution.mdc
Load: .cursor/rules/isolation_rules/Core/notion-memory-bank-ops.mdc
Load: .cursor/rules/isolation_rules/Core/task-creation-notion.mdc
```
(Notion backend: use notion-verification; skip file-verification)

### Step 2: Load VAN Mode Map
```
Load: .cursor/rules/isolation_rules/visual-maps/van_mode_split/van-mode-map.mdc
```

### Step 3: Load Complexity-Specific Rules (Based on Task Analysis)
After determining complexity level, load:
- **Level 1:** `.cursor/rules/isolation_rules/Level1/workflow-level1.mdc`
- **Level 2-4:** Load plan mode rules (transition to PLAN command)

## Workflow

1. **Platform Detection**
   - Detect operating system
   - Adapt commands for platform
   - Set path separators

2. **Task Creation / Task Validation**
   - Read config: projectId, taskId, issueId, issueUrl, tasksDataSourceUrl, projectsDataSourceUrl (`issueId` / `issueUrl` are optional; overwritten from Notion in Step 3 when the Task row has issue fields)
   - Treat `taskId` as "needs creation" when it is `null` or empty string `""`
   - **If taskId is null/empty**: Follow `Core/task-creation-notion.mdc`. User MUST provide task description in `/van [description]`; if missing, ask: "Please provide a task description, e.g. /van Add user authentication feature"
   - **If taskId exists AND user provided description**: notion-fetch current Task page, compare its title with user's van description
     - If description clearly differs from current task (different topic/intent): ask: "Current task (<taskId>: <title>) doesn't match your description. Is this a new task? Should I create it?"
     - If user confirms new task: follow task-creation-notion.mdc, update config
     - If user says no: continue with existing task
   - **Task-scoped subpage URLs when `taskId` changes:** Whenever the primary **`taskId`** becomes a **different** value than before this run (including: task created from null/empty **`taskId`**, or user-confirmed new task after description mismatch), set **`creativePageUrl`**, **`reflectionPageUrl`**, and **`archivePageUrl`** to **`null`** in `.cursor/notion-memory-bank.json` before Memory Bank verification. Those keys refer to pages under a specific Task; keeping old values would point at the wrong task. (**`/change-task`** applies the same rule when switching primary Task ID.) Write the file with `JSON.stringify(obj, null, 2)` when any value changes.

3. **Memory Bank Verification** (MANDATORY – follow **`Core/notion-verification.mdc`** in order; do **not** substitute a shorter checklist that skips steps)
   - [ ] **Step 1** — Read full config from `.cursor/notion-memory-bank.json` (all keys in **`notion-verification.mdc`** step 1, including **`productContextPageUrl`**, **`systemPatternsPageUrl`**, **`techContextPageUrl`**, **`styleGuidePageUrl`**).
   - [ ] **Step 2** — If `projectsDataSourceUrl` / `tasksDataSourceUrl` are missing, placeholders, or invalid `collection://` ids: recover per **`Core/notion-memory-bank-ops.mdc`** (**Recover invalid or missing data source URLs**).
   - [ ] **Step 3** — Task creation / validation: already covered by **this command’s Step 2** above (same content as **`notion-verification.mdc`** step 3 when running `/van`).
   - [ ] **Step 4** — Resolve Project and Task per **`Core/notion-verification.mdc`** + **`Core/notion-task-id-resolution.mdc`**.
   - [ ] **Step 5** — **Stale detection for all six Project Memory Bank subpages:** **`activeContextPageUrl`**, **`progressPageUrl`**, **`productContextPageUrl`**, **`systemPatternsPageUrl`**, **`techContextPageUrl`**, **`styleGuidePageUrl`** — per **`notion-verification.mdc`** step 5 (not only Active Context / Progress). Task subpages (**`creativePageUrl`**, etc.) are still handled by **`/creative`**, **`/reflect`**, **`/archive`** — unchanged.
   - [ ] **Step 6** — **`notion-fetch`** Project and Task; sync **`projectName`**, **`taskName`**, **`issueId`**, **`issueUrl`**, **`projectPageUrl`**, **`taskPageUrl`** per **`notion-verification.mdc`** step 6 (including Task–Project relation fix and **`/van`** Active Context block update rules in that step).
   - [ ] **Step 7** — **Search/create all six Project subpages** when each **`*PageUrl`** is null after step 5: **`Active Context <projectId>`**, **`Progress <projectId>`**, **`Product Context <projectId>`**, **`System Patterns <projectId>`**, **`Tech Context <projectId>`**, **`Style Guide <projectId>`** — exactly as **`notion-verification.mdc`** step 7. Update config with new or resolved **`*PageUrl`** values.
   - [ ] **Step 8** — Confirm Project, Task, and all six Project subpages above are accessible (report per **`notion-verification.mdc`** step 8).

4. **Task Analysis**
   - Use the Task page already resolved in Step 3 (**`Task ID`** match); **`notion-fetch`** again only if you need a fresh body — do **not** use Tasks–database semantic **`notion-search`** on numeric `taskId` as the resolver
   - Analyze task requirements
   - Determine complexity level (1-4)

5. **Route Based on Complexity**
   - **Level 1:** Continue in VAN mode, proceed to implementation
   - **Level 2-4:** Transition to `/plan` command. **STOP here**—do NOT implement; user runs `/plan` then `/build` separately.

6. **Update Memory Bank**
   - notion-update-page on Task page (complexity)
   - notion-update-page on activeContext page (**multi-task safe**). Follow `Core/memory-bank-paths.mdc`: **Subpage body (do not echo page title)**—no `#` that repeats the page title; use `##` sections. **Primary pattern:** `## Task <taskId> — <taskName>` for the resolved Task (title from Step 3 fetch). `notion-fetch` Active Context first; **add or replace only** that task’s section (from its `## Task <taskId>` through the next peer `##`), leaving other `## Task …` blocks untouched. If no such section exists, **append** it (do not delete unrelated sections). Legacy-only `## Current focus (YYYY-MM-DD)` may remain; prefer migrating new content into `## Task <taskId> — …`. If the page still has a redundant leading H1 matching the title, remove it first via `update_content`.

## Usage

Type `/van` followed by your task description or initialization request.

Example:
```
/van Initialize project for adding user authentication feature
```

## Next Steps

- **Level 1 tasks:** Proceed directly to `/build` command
- **Level 2-4 tasks:** VAN completes after Step 6. Tell user to run `/plan` for detailed planning, then `/build` for implementation. Do NOT implement in VAN.

