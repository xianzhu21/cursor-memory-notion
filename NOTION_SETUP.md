# Notion Memory Bank Setup Guide

This project uses **Notion** as the Memory Bank backend. **Projects** and **Tasks** both use numeric ids: **Project ID** and **Task ID** (same convention — no `PROJECT-` / `TASK-` prefixes in values).

## 1. Prerequisites

- Notion MCP (plugin-notion-workspace-notion) installed and configured
- **Projects** and **Tasks** databases in Notion with a bidirectional relation

## 2. Identifier Convention

- **Projects**: `projectId` matches **Project ID** (numbers only — in JSON you may use a number `123` or string `"123"`).
- **Tasks**: `taskId` matches **Task ID** (same style as Project ID — e.g. `588` or `"588"`).

- **Project** resolution: **`Core/notion-verification.mdc`** — **URL** > **title** (same token idea as **Task**; **`projectId`** for duplicate-title tiebreaks).
- **Task** resolution: **`Core/notion-task-id-resolution.mdc`** — **URL** > **title**. Keep **`projectId`** and **`taskId`** in config for **duplicate-title** tiebreaks, **`/change-task`**, and Memory Bank invariants. Prefer **`taskPageUrl`**, **`taskName`**, or a pasted task **Notion `https://…` URL** so the row can be found without many **`notion-fetch`** calls.

## 3. Configure Identifiers

Create or preserve your config:

- **If `.cursor/notion-memory-bank.json` already exists** (e.g. you copied `.cursor` from another project): Use it as-is. It may already contain your `projectId`, `taskId`, or data source URLs. Do not overwrite with the example.
- **If the file does not exist**: Copy from the example:

```bash
cp .cursor/notion-memory-bank.json.example .cursor/notion-memory-bank.json
```

Then edit `.cursor/notion-memory-bank.json` with your values. See `.cursor/notion-memory-bank.json.example` for the schema. The actual config is in `.gitignore` to avoid committing personal Notion workspace data. The example uses **`null`** for every placeholder field—after copying, set at least **`projectId`**, **`projectsDataSourceUrl`**, and **`tasksDataSourceUrl`** (see below) before scoped Notion MCP calls will work.

**Required:**
- `projectId` – numeric project key: JSON **number** or **string** (e.g. `123` or `"123"`) — must match **Project ID** on the Project row (also used to disambiguate duplicate project titles)
- `projectPageUrl`, `taskPageUrl`, `projectName`, `taskName` – **strongly recommended**: browse **URLs** (cheapest), else exact **titles** for the **URL** > **title** resolution in Core rules. **`projectId`** / **`taskId`** disambiguate **duplicate titles** and stay in config for invariants.
- `tasksDataSourceUrl`, `projectsDataSourceUrl` – **`collection://…`** strings from `notion-fetch` on the databases (see below). Scoped Notion MCP tools need these; they are **not** normal browser links.

**Optional:**
- `taskId` – numeric task key: JSON **number** or **string** (e.g. `588` or `"588"`) — must match **Task ID** on the Task row. Leave as `null` or empty string `""` to have `/van [task description]` create a new task automatically (same flow as a **file-based Memory Bank**).
- `issueId`, `issueUrl` – usually left `null`; **`/van`** (notion-verification) fills them from the Task row’s **Issue ID** / **Issue URL** properties when you fetch a task (e.g. Task ID `1430`). Override manually in JSON if needed.
- `projectPageUrl`, `taskPageUrl` – optional **`https://`…** Notion links to the resolved Project and Task pages; notion-verification fills them on fetch so you can **Ctrl/Cmd+click** the URL inside the JSON file to open Notion in the browser.
- Subpage fields (`activeContextPageUrl`, `progressPageUrl`, etc.) – usually **`null`** until subpages exist; when set, use full **`https://`…** Notion page URLs (from the browser address bar) so the JSON stays Ctrl/Cmd+clickable. The example file uses **`null`** until agents or you paste links after first fetch.

**Data source URLs:** Fetch the Projects and Tasks databases with `notion-fetch` (using each database’s **`https://`…** URL from the browser or search results) to get the `collection://` URLs from `<data-source>` tags. Use those strings for `projectsDataSourceUrl` and `tasksDataSourceUrl`.

## 4. Mapping (config → Notion)

| cursor-memory-bank | Notion | Config Key |
|--------------------|--------|------------|
| tasks.md | Task page body (plan, checklist) | taskId |
| projectbrief.md | Project page body | projectId |
| activeContext.md | Active Context subpage under Project | activeContextPageUrl |
| progress.md | Progress subpage under Project | progressPageUrl |
| productContext.md, systemPatterns.md, techContext.md | Matching Project subpages (Level 3–4 docs) | productContextPageUrl, systemPatternsPageUrl, techContextPageUrl |
| style-guide.md | Style Guide subpage under Project | styleGuidePageUrl |
| creative / reflection / archive (paths under Task) | Creative, Reflection, Archive subpages under Task | creativePageUrl, reflectionPageUrl, archivePageUrl |

**Parallel tasks:** Config still has **one** `taskId` (primary task for `/van` and task-scoped logs). The **Active Context** page may list **multiple** in-flight tasks using merge-safe `## Task <taskId> — <taskName>` sections—see `Core/memory-bank-paths.mdc` (**Active Context: multiple in-flight tasks**).

## 5. notion-search Lookup

`notion-search` is **semantic search**, not exact property match. **Do not** use **`data_source_url`** = Projects DB with **`query`** = bare numeric **`projectId`**, or **`data_source_url`** = Tasks DB with **`query`** = bare numeric **`taskId`**, as the **sole** way to pick a row—use **URL** and **name** resolution in the Core rules first.

- **Improve accuracy**: Prefer **project** / **task titles** in config (**`projectName`**, **`taskName`**) for the first lookup under the rules above; confirm **Project ID** / **Task ID** on **`notion-fetch`** results. After the first successful resolve, store **`projectPageUrl`** / **`taskPageUrl`** and reuse **`notion-fetch`** on those URLs to avoid repeat search.

## 6. Command Usage

Workflow matches a **file-based Memory Bank** but is **complexity-dependent** (see [README — Complexity Levels](README.md#complexity-levels)):

| Level | Command chain |
|-------|-----------------|
| 1 | `/van` → `/build` → `/reflect` → `/archive` |
| 2 | `/van` → `/plan` → `/build` → `/reflect` → `/archive` |
| 3–4 | `/van` → `/plan` → `/creative` → `/build` → `/reflect` → `/archive` |

The **longest** path (Levels 3–4) is: `/van` → `/plan` → `/creative` → `/build` → `/reflect` → `/archive`.

**First run:** Set `taskId` to `null` in config. Run `/van Add user authentication to the application` – a new task is created in Notion and `taskId` is updated automatically.

**Switch task (existing Task ID):** **`/change-task <Task ID>`** updates config and runs verification. Include a **Notion task page `https://…` URL** in the same message or set **`taskName`** in config to the **exact** title when **`taskPageUrl`** was cleared—per **`Core/notion-task-id-resolution.mdc`**. Alternatively set `taskId` in `.cursor/notion-memory-bank.json` and run **`/van`** (with a description line, URL, or ID) for full init. For a **new** task from description, use **`/van [description]`** (or clear `taskId` first per task-creation flow).

## 7. Token Optimization

- **Read**: Fetch only when needed; prefer once per session.
- **Write**: On explicit trigger or at command end.

## 8. Invalid or outdated `collection://` URLs

If `projectsDataSourceUrl` or `tasksDataSourceUrl` is wrong (copied from another workspace, placeholder, or MCP errors when searching scoped to that URL):

1. In Notion, locate your **Projects** and **Tasks** databases (sidebar or workspace search).
2. Open each database in the browser and copy its URL, **or** use Notion MCP workspace search to find databases by title.
3. Run **`notion-fetch`** on each database URL. From the response, copy the value inside **`<data-source url="collection://...">`** — that full `collection://...` string is what belongs in config.
4. Update `.cursor/notion-memory-bank.json` with `JSON.stringify(..., null, 2)` formatting.

When an agent runs **notion-verification** (e.g. during `/van`), it should follow the same recovery flow automatically if URLs are missing or invalid — see `Core/notion-memory-bank-ops.mdc` (**Recover invalid or missing data source URLs**).

## 9. Migration (Existing Projects)

If you have existing subpages with old names (`Active Context`, `Progress` without projectId; `Creative`, `Reflection`, `Archive` without taskId):

- **Option A (recommended)**: Clear `activeContextPageUrl` and `progressPageUrl` in config, then run `/van`. New pages will be created with correct names (`Active Context <projectId>`, `Progress <projectId>` — e.g. `Active Context 37`). For Task subpages: clear `creativePageUrl`, `reflectionPageUrl`, and `archivePageUrl` if desired; new pages will be created with correct names when you run `/creative`, `/reflect`, or `/archive`. You can delete or archive the old pages in Notion.
- **Option B**: Manually rename pages in Notion to match the new convention, then update config if needed. Page IDs stay the same; only titles change.
