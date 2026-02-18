# Notion Memory Bank Setup Guide

This project uses **Notion** as the Memory Bank backend. Projects and Tasks are identified by **PROJECT-** and **TASK-** prefixes.

## 1. Prerequisites

- Notion MCP (plugin-notion-workspace-notion) installed and configured
- **Projects** and **Tasks** databases in Notion with a bidirectional relation

## 2. Identifier Convention

- **PROJECT-** prefix: Look up in **Projects** database
- **TASK-** prefix: Look up in **Tasks** database

Example: `PROJECT-123` → search Projects database; `TASK-588` → search Tasks database.

## 3. Configure Identifiers

Create or preserve your config:

- **If `.cursor/notion-memory-bank.json` already exists** (e.g. you copied `.cursor` from another project): Use it as-is. It may already contain your `projectId`, `taskId`, or data source URLs. Do not overwrite with the example.
- **If the file does not exist**: Copy from the example:

```bash
cp .cursor/notion-memory-bank.json.example .cursor/notion-memory-bank.json
```

Then edit `.cursor/notion-memory-bank.json` with your values. See `.cursor/notion-memory-bank.json.example` for the schema. The actual config is in `.gitignore` to avoid committing personal Notion workspace data.

**Required:**
- `projectId` – e.g. `PROJECT-123` (your Project)
- `taskId` – e.g. `TASK-588` (your current Task)

**Data source URLs:** Fetch the Projects and Tasks databases with `notion-fetch` to get the `collection://` URLs from `<data-source>` tags. Use these for `projectsDataSourceUrl` and `tasksDataSourceUrl`.

**Optional:** Set subpage IDs if you've already created them. Leave as `null` to have commands create them automatically.

## 4. Mapping (by ID)

| cursor-memory-bank | Notion | Config Key |
|--------------------|--------|------------|
| tasks.md | Task page body (plan, checklist) | taskId |
| projectbrief.md | Project page body | projectId |
| activeContext.md | activeContext subpage under Project | activeContextPageId |
| progress.md | progress subpage under Project | progressPageId |
| creative/reflection/archive | Creative, Reflection, Archive subpages under Task | creativePageId, reflectionPageId, archivePageId |

## 5. notion-search Lookup

`notion-search` is **semantic search**, not exact property match. When resolving `PROJECT-X` or `TASK-Y`:

- **Improve accuracy**: Add an ID property (e.g. `userDefined:ID`) to your Projects and Tasks databases, and set values like `PROJECT-37`, `TASK-1381`. Semantic search is more likely to return the correct page when the ID appears in page content or properties.
- **If results are wrong**: After the first successful fetch, you can store the resolved page URL or ID in config (e.g. as a cached value) and use `notion-fetch` directly with that ID for subsequent operations, bypassing search.

## 6. Command Usage

Same as before: `/van` → `/plan` → `/creative` → `/build` → `/reflect` → `/archive`

Update `taskId` when you switch to a different task.

## 7. Token Optimization

- **Read**: Fetch only when needed; prefer once per session.
- **Write**: On explicit trigger or at command end.
