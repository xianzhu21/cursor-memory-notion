# CREATE-TASK Command - New Task in Notion

This command creates a new task in the Tasks database and associates it with the current project. It uses project config and invokes the create-task workflow with pre-resolved context.

## Memory Bank Integration (Notion)

Reads from `.cursor/notion-memory-bank.json`:
- `tasksDataSourceUrl` – Tasks database data source (e.g. `collection://d1b195db-550c-411f-84b8-8067fe4a12c3`)
- `projectId` – Current project (e.g. `PROJECT-37`)
- `projectsDataSourceUrl` – For resolving projectId

Uses `notion-create-pages` to create the task row. Optionally updates `taskId` after creation.

## Rule Loading (Optional)

For Notion resolution, load:
```
Load: .cursor/rules/isolation_rules/Core/memory-bank-paths.mdc
```

## Workflow

1. **Load Config**
   - Read `.cursor/notion-memory-bank.json`
   - Extract `tasksDataSourceUrl`, `projectId`, `projectsDataSourceUrl`

2. **Resolve Project**
   - Use `notion-search` with `data_source_url` = `projectsDataSourceUrl`, `query` = `projectId` (e.g. `PROJECT-37`)
   - Get project page URL from first result (needed for Project relation)

3. **Gather Task Info**
   - If user did not provide task title in the command, ask: "Task 标题（必填）？"
   - Optional fields to ask if not provided: Status, Assignee, Dates/Due, Module, Man-Day, 正文
   - Status options: `Not Started`, `Pending`, `In Progress`, `Done`
   - Assignee options: `Lichen`, `Nengfei`, `Xianzhu`, `Zhuoyi`

4. **Create Task**
   - Use `notion-create-pages` with:
     - `parent`: `{ "data_source_id": "<uuid from tasksDataSourceUrl>" }` (extract UUID from `collection://...`)
     - `pages`: `[{ "properties": { "Name": "<title>", "Project": "<project page URL>", "Status": "<status>", ... }, "content": "<optional markdown>" }]`
   - Extract `data_source_id` from `tasksDataSourceUrl` (the UUID after `collection://`)
   - For date properties use: `date:Due:start`, `date:Due:is_datetime` (0), etc.
   - For relation `Project`: pass project page URL as **string** (not array)

5. **Confirm & Return TASK-ID**
   - Get TASK-ID: `notion-fetch` the newly created page (from create response URL/id), read `userDefined:ID` (e.g. `TASK-1382`)
   - **Return prominently**: `TASK-ID: TASK-xxxx`
   - Also return: task title, key properties, Notion link
   - Ask: "是否将 `taskId` 更新为新创建的 task？" – if yes, update `.cursor/notion-memory-bank.json` with the new TASK- identifier

## Skill Reference

Follow the create-task skill for property mapping and validation. This command provides:
- **Tasks database**: use `tasksDataSourceUrl` directly (no search)
- **Project**: use resolved `projectId` page (always associate)

## Usage

```
/create-task <task title>
```

Or without title – the AI will ask:

```
/create-task
```

Example:
```
/create-task 实现 Notion 同步 API 重试逻辑
```

## Next Steps

After creation, use `/van` to load the new task context, or `/plan` if planning is needed.
