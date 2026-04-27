# VAN Command - Initialization & Entry Point

This command initializes the Memory Bank system, performs platform detection, determines task complexity, and routes to appropriate workflows.

To **switch** the config primary task to another **existing** Task ID and **load** that task’s progress **without** this full init (no complexity step, no whole-page Progress rewrite), use **`/change-task`** instead.

**MANDATORY: When /van is invoked, execute the full workflow** (Steps 1–6) **unless** the **Stop and ask** gate in **VAN: Resolution from the user message vs config** fires—in that case **stop immediately** after the gate with the ask only (**do not** run Workflow Steps 3–6 or Memory Bank verification). The "full workflow" means **only** these 6 initialization steps—NOT implementation of the user's task. Do NOT skip steps when the gate passes. Do NOT substitute with other actions (e.g. only updating rules or files) even if the user message includes additional text. If the user message contains text that could be a task description (e.g. "Add X", "Clarify Y", "明确 Z"), treat it as the `/van [description]` and run Task Creation/Validation accordingly.

**CRITICAL for Level 2-4:** VAN **never** implements code, rules, or docs. For Level 2-4 tasks, VAN **stops** after Step 6 and hands off to `/plan`. Implementation is done in `/build`, not in VAN.

## VAN: Resolution from the user message vs config

**Message anchors** (then config): **Notion `https://`…** URLs, **name or description** (e.g. `/van …` body), and **numeric IDs** to merge into JSON (**Task ID** / **Project ID**). **Row resolution** is **URL** > **title** in **`Core/notion-verification.mdc`** (Project) and **`notion-task-id-resolution.mdc`** (Task); **IDs** in config are for **duplicate-title** tiebreaks and invariants, not an **ID**-only database lookup. Take hints **from the user message first**, then **`.cursor/notion-memory-bank.json`**.

**Priority when the prompt supplies multiple hints** (token-efficient: **URL** > **title** for **Project** and **Task**; strongest for **that** entity wins):

| Entity | Order (strongest first) |
|--------|-------------------------|
| **Task** | Notion **task** page **`https://`…** URL → **title or description** (including `/van` text) → (optional) **Task ID** to merge with config — details in **`Core/notion-task-id-resolution.mdc`** |
| **Project** | Notion **project** page **`https://`…** URL → **project title / name** in the message → merge with config **`projectPageUrl`**, **`projectName`**, **`projectId`** — details in **`Core/notion-verification.mdc`** step 4 |

- **`notion-fetch`** a **prompt-supplied Task or Project URL** when present to anchor the row, then sync identifiers into config per **`Core/notion-verification.mdc`** / **`Core/notion-memory-bank-ops.mdc`**.
- After anchors are fixed, continue with **`Core/notion-verification.mdc`** + **`Core/notion-task-id-resolution.mdc`** as usual (search under project, relation lists, duplicate-title rules, etc.).

**Stop and ask the user** (do **not** start **Memory Bank Verification** / **Workflow** Step 3) when the **user message** for this `/van` run provides **no specific task information**—**regardless** of whether config **`taskId`** is already set. Config alone is **not** enough; the prompt must anchor the task for this run.

**Specific task information** means **at least one** of (per **`Core/notion-task-id-resolution.mdc`**; **ID** in the message updates **config** only):

1. A Notion **`https://`…** URL that identifies a **Task** page, **or**
2. A **non-empty** line of text after the **`/van`** token (task **description** or title intent—trim whitespace; slash-command only with no trailing text **does not** count), **or**
3. An explicit **Notion Task ID** the user intends for this run (a bare number counts only if clearly the Task ID, e.g. labeled or in a `/van 1430`-style invocation—not an unrelated number such as a date).

Reply with a single clear ask, e.g. *Add a task line: `/van <description>`*, *paste the task Notion URL*, or *give the Notion **Task ID** (or use **`/change-task <id>`** to switch without re-describing the task).*

**Routine re-init on the same task without new prompt text:** use **`/change-task`** (same **`taskId`**) or send **`/van`** again **with** one of the items above—not a bare **`/van`**.

## Memory Bank Integration (Notion)

**CRITICAL:** This project uses **Notion** as Memory Bank. Use **`*PageUrl`** values (or **`projectPageUrl` / `taskPageUrl`**) from `.cursor/notion-memory-bank.json`—full **`https://`…** Notion page URLs for `notion-fetch` / updates (agents derive internal create-parent fields from those URLs in-session only; they are not stored as bare ids in JSON). **Also** apply **VAN: Resolution from the user message vs config** above when parsing the same user message:
- **tasks** - Task page body: **`Core/notion-task-id-resolution.mdc`** — **`taskPageUrl`**, then **`taskName`** (or **issue** keyword) + project-scoped **`notion-search`** + **candidate** fetches. **Duplicate titles** among candidates → disambiguate with **`taskId`**. Do **not** use Tasks–DB **`notion-search`** on a bare numeric id as the only step.
- **issueId** / **issueUrl** - External tracker key and browse URL; **not** the Notion **Task ID**. Usually read from the Task’s **Issue ID** **text** property (often `[KEY](https://...)` in one field); `issueUrl` is parsed from that link unless the database has a separate URL column. Synced when notion-verification runs — same Task fetch as `taskName`.
- **activeContext** - activeContext page (`activeContextPageUrl`). May contain **multiple** parallel tasks; see `Core/memory-bank-paths.mdc` **Active Context: multiple in-flight tasks**. `/van` updates **only** the section for config **`taskId`**.
- **progress** - progress page (`progressPageUrl`)
- **projectBrief** - Project page body: **`Core/notion-verification.mdc`** step 4 — **`projectPageUrl`**, then **`projectName`** (**URL** > **title**); **duplicate project titles** → **Project ID** / **`projectId`** tiebreaker.

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
   - Read config: projectId, projectName, projectPageUrl, taskId, taskName, taskPageUrl, issueId, issueUrl, tasksDataSourceUrl, projectsDataSourceUrl (`issueId` / `issueUrl` are optional; overwritten from Notion in Step 3 when the Task row has issue fields)
   - **Parse the user message** per **VAN: Resolution from the user message vs config** (**URL** / **title** / **prompt id** → merge with config; **Task** row resolution = **URL** > **title** per **`notion-task-id-resolution.mdc`**).
   - **Prompt URLs / IDs:** If the message contains a Notion **`https://`…** URL for a **Task** page, **`notion-fetch`** it and write **`taskPageUrl`**, **`taskId`**, **`taskName`** (and **`issueId` / `issueUrl`** when present) into config when they differ from the fetch—**before** Step 3. Same for a **Project** page URL → **`projectPageUrl`**, **`projectId`**, **`projectName`**. If the message gives an explicit **Task ID** or **Project ID** without a URL, set that field in config for this run (then verify in Step 3).
   - **Gate — stop and ask** if the prompt lacks **specific task information** (see **Stop and ask** in **VAN: Resolution from the user message vs config**)—**even when** config **`taskId`** is set. **Do not** proceed to Step 3 until a **new** user message supplies it.
   - Treat `taskId` as "needs creation" when it is `null` or empty string `""`
   - **If taskId is null/empty** and the gate passed: Follow `Core/task-creation-notion.mdc` using the **description** from `/van …` or equivalent prompt text
   - **If taskId exists AND user provided description**: notion-fetch current Task page, compare its title with user's van description
     - If description clearly differs from current task (different topic/intent): ask: "Current task (<taskId>: <title>) doesn't match your description. Is this a new task? Should I create it?"
     - If user confirms new task: follow task-creation-notion.mdc, update config
     - If user says no: continue with existing task
   - **Task-scoped URLs and name when `taskId` changes:** Whenever the primary **`taskId`** becomes a **different** value than before this run (including: task created from null/empty **`taskId`**, or user-confirmed new task after description mismatch), set **`taskPageUrl`**, **`taskName`**, **`creativePageUrl`**, **`reflectionPageUrl`**, and **`archivePageUrl`** to **`null`** in `.cursor/notion-memory-bank.json` before Memory Bank verification. Stale **URL** or **name** can mis-resolve under **URL** > **title** resolution. (**`/change-task`** clears **`taskPageUrl`** and **`taskName`** when switching Task ID.) Write the file with `JSON.stringify(obj, null, 2)` when any value changes.

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
   - Use the Task page already resolved in Step 3 (**`notion-task-id-resolution`**); **`notion-fetch`** again only if you need a fresh body — do **not** use Tasks–database semantic **`notion-search`** on numeric `taskId` as the resolver
   - Analyze task requirements
   - Determine complexity level (1-4)

5. **Route Based on Complexity**
   - **Level 1:** Continue in VAN mode, proceed to implementation
   - **Level 2-4:** Transition to `/plan` command. **STOP here**—do NOT implement; user runs `/plan` then `/build` separately.

6. **Update Memory Bank**
   - notion-update-page on Task page (complexity)
   - notion-update-page on activeContext page (**multi-task safe**). Follow `Core/memory-bank-paths.mdc`: **Subpage body (do not echo page title)**—no `#` that repeats the page title; use `##` sections. **Primary pattern:** `## Task <taskId> — <taskName>` for the resolved Task (title from Step 3 fetch). `notion-fetch` Active Context first; **add or replace only** that task’s section (from its `## Task <taskId>` through the next peer `##`), leaving other `## Task …` blocks untouched. If no such section exists, **append** it (do not delete unrelated sections). Legacy-only `## Current focus (YYYY-MM-DD)` may remain; prefer migrating new content into `## Task <taskId> — …`. If the page still has a redundant leading H1 matching the title, remove it first via `update_content`.

## Usage

**`/van` alone is invalid**—always include **task description text**, a **task Notion URL**, or an explicit **Task ID** in the same message. To reload the current task without new description, use **`/change-task`** (with the same id) instead of a bare **`/van`**.

Example:
```
/van Initialize project for adding user authentication feature
```

## Next Steps

- **Level 1 tasks:** Proceed directly to `/build` command
- **Level 2-4 tasks:** VAN completes after Step 6. Tell user to run `/plan` for detailed planning, then `/build` for implementation. Do NOT implement in VAN.

