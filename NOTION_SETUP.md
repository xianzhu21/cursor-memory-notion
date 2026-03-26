# Notion Memory Bank Setup Guide

This project uses **Notion** as the Memory Bank backend. **Projects** and **Tasks** both use numeric ids: **Project ID** and **Task ID** (same convention — no `PROJECT-` / `TASK-` prefixes in values).

## 1. Prerequisites

- Notion MCP (plugin-notion-workspace-notion) installed and configured
- **Projects** and **Tasks** databases in Notion with a bidirectional relation

## 2. Identifier Convention

- **Projects**: `projectId` matches **Project ID** (numbers only — in JSON you may use a number `123` or string `"123"`).
- **Tasks**: `taskId` matches **Task ID** (same style as Project ID — e.g. `588` or `"588"`).

Example: `projectId` `123` or `"123"` → **Projects** by **Project ID**; `taskId` `588` or `"588"` → **Tasks** by **Task ID**.

## 3. Configure Identifiers

Create or preserve your config:

- **If `.cursor/notion-memory-bank.json` already exists** (e.g. you copied `.cursor` from another project): Use it as-is. It may already contain your `projectId`, `taskId`, or data source URLs. Do not overwrite with the example.
- **If the file does not exist**: Copy from the example:

```bash
cp .cursor/notion-memory-bank.json.example .cursor/notion-memory-bank.json
```

Then edit `.cursor/notion-memory-bank.json` with your values. See `.cursor/notion-memory-bank.json.example` for the schema. The actual config is in `.gitignore` to avoid committing personal Notion workspace data.

**Required:**
- `projectId` – numeric project key: JSON **number** or **string** (e.g. `123` or `"123"`) — must match **Project ID** on the Project row
- `tasksDataSourceUrl`, `projectsDataSourceUrl` – from `notion-fetch` on the databases (see below)

**Optional:**
- `taskId` – numeric task key: JSON **number** or **string** (e.g. `588` or `"588"`) — must match **Task ID** on the Task row. Leave as `null` or empty string `""` to have `/van [task description]` create a new task automatically (same flow as original cursor-memory-bank).
- Subpage IDs (`activeContextPageId`, `progressPageId`, etc.) – leave as `null` to have commands create them automatically.

**Data source URLs:** Fetch the Projects and Tasks databases with `notion-fetch` to get the `collection://` URLs from `<data-source>` tags. Use these for `projectsDataSourceUrl` and `tasksDataSourceUrl`.

## 4. Mapping (by ID)

| cursor-memory-bank | Notion | Config Key |
|--------------------|--------|------------|
| tasks.md | Task page body (plan, checklist) | taskId |
| projectbrief.md | Project page body | projectId |
| activeContext.md | activeContext subpage under Project | activeContextPageId |
| progress.md | progress subpage under Project | progressPageId |
| creative/reflection/archive | Creative, Reflection, Archive subpages under Task | creativePageId, reflectionPageId, archivePageId |

## 5. notion-search Lookup

`notion-search` is **semantic search**, not exact property match. When resolving `projectId` or `taskId`:

- **Improve accuracy**: Use **Project ID** on **Projects** and **Task ID** on **Tasks** (numeric values). After resolving, verify MCP properties: `userDefined:Project ID` and `userDefined:Task ID`.
- **If results are wrong**: After the first successful fetch, you can store the resolved page URL or ID in config (e.g. as a cached value) and use `notion-fetch` directly with that ID for subsequent operations, bypassing search.

## 6. Command Usage

Same as original cursor-memory-bank: `/van` → `/plan` → `/creative` → `/build` → `/reflect` → `/archive`

**First run:** Set `taskId` to `null` in config. Run `/van Add user authentication to the application` – a new task is created in Notion and `taskId` is updated automatically.

**Switch task:** Use `/create-task <title>` to create a new task, then run `/van` (or update `taskId` manually and run `/van`).

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

- **Option A (recommended)**: Clear `activeContextPageId` and `progressPageId` in config, then run `/van`. New pages will be created with correct names (`Active Context <projectId>`, `Progress <projectId>` — e.g. `Active Context 37`). For Task subpages: clear `creativePageId`, `reflectionPageId`, and `archivePageId` if desired; new pages will be created with correct names when you run `/creative`, `/reflect`, or `/archive`. You can delete or archive the old pages in Notion.
- **Option B**: Manually rename pages in Notion to match the new convention, then update config if needed. Page IDs stay the same; only titles change.
