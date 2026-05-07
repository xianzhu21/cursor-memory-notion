# CHANGE-TASK Command — Switch primary task and load its progress

Use this command when you already have a **Task ID** in Notion and want to **switch the Memory Bank primary task** without running the full **`/van`** initialization (no mandatory platform pass, **no** new-task creation flow, **no** complexity analysis or handoff to **`/plan`**).

**MANDATORY: When `/change-task` is invoked, execute the workflow below in order.** Do not substitute **`/van`**. Do not implement the user’s product work—only context switch and verification/summary.

## When to use vs `/van`

| Goal | Command |
|------|---------|
| Switch `taskId`, sync names/issue fields, **show this task’s saved state** in chat | **`/change-task`** |
| First entry, **create** a task from description, complexity, routing, full Step 6 Memory Bank init | **`/van`** |

## Memory Bank integration (Notion)

**CRITICAL:** Read and write **`.cursor/notion-memory-bank.json`** with **`JSON.stringify(obj, null, 2)`** (2-space indent). Resolve the Task with **`Core/notion-task-id-resolution.mdc`**: **`taskPageUrl`**, then **`taskName`** / **issue** keyword (**candidate** fetches). After a **`taskId`** switch, **`taskPageUrl`** and **`taskName`** are nulled (step 3)—if verification cannot resolve, follow that file’s **incomplete** ask (e.g. paste a **Notion task `https://…` URL** or set **`taskName`** to the **exact** title). Do **not** use semantic **`notion-search`** on the Tasks data source with a bare numeric id as the only proof.

## Progressive rule loading

### Step A — Core rules (always)

```
Load: .cursor/rules/isolation_rules/main.mdc
Load: .cursor/rules/isolation_rules/Core/memory-bank-paths.mdc
Load: .cursor/rules/isolation_rules/Core/notion-retry.mdc
Load: .cursor/rules/isolation_rules/Core/notion-verification.mdc
Load: .cursor/rules/isolation_rules/Core/notion-task-id-resolution.mdc
Load: .cursor/rules/isolation_rules/Core/notion-memory-bank-ops.mdc
```

Do **not** load **`/van`** mode maps, **`task-creation-notion.mdc`** (unless you must point the user to **`/van`** because `taskId` is missing), or Level 1–4 workflow routing rules.

## Workflow

1. **Resolve target Task ID**
   - Accept **`/change-task <id>`** or a clear numeric **Task ID** in the user message. **Recommended:** a **Notion task page `https://`…** URL in the same message, or the **exact** task **title**, so the title tier can resolve after URLs are cleared.
   - If **no Task ID** is given: ask once for the **Notion Task ID** (the value of the **Task ID** property). **Do not** invent an id. **Do not** create a new task here—if `taskId` is `null`/`""` and the user wants a **new** row, tell them to run **`/van [description]`**.

2. **Read config and remember previous primary task (optional)**
   - Read **`.cursor/notion-memory-bank.json`**: same key order as **`.cursor/notion-memory-bank.json.example`** — `taskId` (old), `taskName`, `taskPageUrl`, `issueId`, `issueUrl`, `creativePageUrl`, `reflectionPageUrl`, `archivePageUrl`, `projectId`, `projectName`, `projectPageUrl`, project subpage **`*PageUrl`** keys, `projectsDataSourceUrl`, `tasksDataSourceUrl`.
   - Normalize old vs new **`taskId`** (string/number). If the new id equals the current primary **`taskId`** (after normalization), still run verification and **refresh** the loaded summary—skip unnecessary config writes.

3. **Write new primary `taskId`**
   - Merge into config: set **`taskId`** to the user’s target (JSON number or string is fine per project rules).
   - **Task-scoped URLs and name when the primary task changed:** If the normalized **new `taskId` ≠ old `taskId`**, set **`taskPageUrl`**, **`taskName`**, **`creativePageUrl`**, **`reflectionPageUrl`**, and **`archivePageUrl`** to **`null`** in config (stale **URL** or **name** mis-resolves under **URL** > **title** resolution for the new id). Write the file only when values change.

4. **Notion verification (subset of `Core/notion-verification.mdc`)**
   Execute **steps 1, 2, 4, 5, 6 (partial), 7** of **`notion-verification.mdc`** with these constraints:
   - **Skip** verification **step 3** entirely (that step is **`/van`–only** task creation / description mismatch).
   - **Step 4–7:** Same as verification: resolve **Project** and **Task**, recover bad **`collection://`** URLs if needed, **stale-check** project subpages (**activeContext**, **progress**, etc.), **fetch** Project and Task pages, **sync** `projectName`, `taskName`, `issueId`, `issueUrl`, `projectPageUrl`, `taskPageUrl` from Notion into config when they differ.
   - **Step 6 — do *not* do `van` Step 6 writes:** Do **not** update the Task page **complexity** property. Do **not** run **`/van`** routing or checklist.
   - **Progress page:** Do **not** use **`replace_content`** on **`Progress <projectId>`** for this command. **Do not** “reset” or rewrite the whole Progress narrative to match the new task. If verification’s general reconciliation would require a **large** Progress rewrite, **stop** at summarizing in chat instead (see step 5). If a **small** merge-safe fix is clearly needed for **this** `taskId` only (e.g. one line in a multi-task list), you **may** use **`update_content`** with targeted `old_str` / `new_str` per **`Core/memory-bank-paths.mdc`**.
   - **Active Context:** If you update Notion, follow **`Core/memory-bank-paths.mdc`** **Active Context: multiple in-flight tasks**: **add or update only** the **`## Task <taskId> — <taskName>`** block for the **new** primary task; **preserve** all other **`## Task …`** sections. Prefer **`update_content`**. If the block is missing, **append** it (do not clear the page).

5. **Load task progress (MANDATORY deliverable in chat)**
   After the Task page is resolved and fetched, present a **structured English summary** (headings/bullets) so the user can continue work without re-running **`/van`**:
   - **Task:** `<taskId>`, title, **Status** (and other key Task properties if present in the fetch).
   - **Issue:** `issueId` / `issueUrl` from config after sync.
   - **Task page body:** Short extract of plan / checklist / **## VAN** / implementation notes—enough to reflect **where** the task left off (e.g. build completed).
   - **Active Context:** If the page has **`## Task <taskId> — …`**, quote or summarize that section.
   - **Optional reads:** If **`creativePageUrl`** / **`reflectionPageUrl`** are non-null **after** resolution under the new task, **`notion-fetch`** those pages and add a **brief** note (headings only or 2–3 bullets each). If still null, state that no Creative/Reflection **`*PageUrl`** values are configured for this task yet.

6. **Re-resolve task subpage URLs (populate config when safe)**
   - Under the resolved **Task** page, discover children titled **`Creative <taskId>`** / **`Reflection <taskId>`** (same title pattern as **`Core/memory-bank-paths.mdc`**). If found and **parent** is the resolved Task page, set **`creativePageUrl`** / **`reflectionPageUrl`** in config. If not found, leave **`null`** ( **`/creative`** / **`/reflect`** may create them later).

7. **Close out**
   - Confirm the updated **primary** `taskId` and that config was written with formatted JSON.
   - Tell the user to use **`/van`** when they need **full** initialization (new task, complexity, routing), **`/plan`** / **`/build`** etc. when continuing implementation—not **`/change-task`**.

## Usage

```
/change-task 1430
```

```
/change-task
1430
```

(Second form: user sends the id on the next line or in the same message.)

## Next steps

- Continue implementation: **`/build`** (and **`/plan`** / **`/creative`** as needed for that task’s complexity).
- New task or full init: **`/van`**.
