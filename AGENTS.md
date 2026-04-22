# AGENTS.md

## Learned User Preferences

- When writing `.cursor/notion-memory-bank.json`, use `JSON.stringify(obj, null, 2)`; never minified single-line JSON
- After each notion write: output the modified content in chat (you have it—no notion-fetch). Follow notion-output-after-write skill for Original → Modified format
- For Markdown the user reads in chat (samples, verification blocks, structured notes), follow `.cursor/rules/user/chat-markdown-preview.mdc`: preview-first body text, not outer ` ```markdown ` fences that hide rendering

## Learned Workspace Facts

- `projectName`, `taskName`, `issueId`, `issueUrl`, `projectPageUrl`, and `taskPageUrl` in config are auto-populated when notion-verification fetches Project/Task pages; **Issue ID** is usually **text** (often `[KEY](https://...)`), so `issueUrl` is typically parsed from that string, not a separate Notion column. **`projectPageUrl` / `taskPageUrl`** hold normal **`https://`…** Notion links so you can Ctrl/Cmd+click them inside `.cursor/notion-memory-bank.json` in the editor
- This project uses Notion as Memory Bank backend; config stores **`*PageUrl`** / browse fields as full **`https://`…** page URLs, names, and **`collection://`** data sources for quick reference
- Projects use **Project ID** (numeric); Tasks use **Task ID** (numeric, same convention); `projectId` / `taskId` in config may be JSON **numbers** or **strings** (e.g. `123` or `"123"`)
- If `projectsDataSourceUrl` / `tasksDataSourceUrl` are invalid: find Projects/Tasks databases via Notion search → `notion-fetch` → read `<data-source url="collection://...">` → update config (see `Core/notion-memory-bank-ops.mdc`). **`collection://`** values are required for MCP scoped tools
- Memory Bank subpages in config use **`*PageUrl`** keys (e.g. **`activeContextPageUrl`**, **`creativePageUrl`**); legacy `activeContextPageId`-style keys should be renamed to the matching **`*PageUrl`** names (same string values)
- Notion Memory Bank **Project subpage bodies** (activeContext, progress, etc.): do not repeat the page title as `#` in the body; Notion already shows the title—start sections at `##` (see `Core/memory-bank-paths.mdc` **Subpage body (do not echo page title)**)
- **Active Context** may track **several** parallel tasks via `## Task <taskId> — …` sections; `/van` updates **only** the block for config `taskId`; **`/archive` removes** that task’s section from Active Context (see **Active Context: multiple in-flight tasks** in `Core/memory-bank-paths.mdc`)
- Treat numeric **`projectId`** and **`taskId`** as authoritative over **`projectName`** / **`taskName`** when resolving config or matching Notion rows (same priority idea for Project and Task)
- To switch the primary in-flight task and **load** its saved state **without** full **`/van`** init (avoids complexity routing and whole-page Progress reset), use **`/change-task`**; use **`/van`** for full initialization, new-task routing, or when you explicitly need that workflow
- When the primary **`taskId`** changes (`/change-task`, or new/confirmed task during `/van` Step 2), set **`creativePageUrl`**, **`reflectionPageUrl`**, and **`archivePageUrl`** to **`null`** in config so those values do not point at the previous task’s subpages
- **`/archive`** updates Notion pages; it does **not** wholesale-reset **`.cursor/notion-memory-bank.json`** or clear **`taskId`**. For the next task, use **`/van`** (including new task) or **`/change-task`** (switch only)
