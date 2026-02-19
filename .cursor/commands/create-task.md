# CREATE-TASK Command - New Task in Notion

This command creates a new task in the Tasks database and associates it with the current project. It uses project config and invokes the create-task workflow with pre-resolved context.

## Memory Bank Integration (Notion)

Reads from `.cursor/notion-memory-bank.json`:
- `tasksDataSourceUrl` – Tasks database data source (e.g. `collection://d1b195db-550c-411f-84b8-8067fe4a12c3`)
- `projectId` – Current project (e.g. `PROJECT-37`)
- `projectsDataSourceUrl` – For resolving projectId

Uses `notion-create-pages` to create the task row. Optionally updates `taskId` after creation.

## Rule Loading

```
Load: .cursor/rules/isolation_rules/Core/memory-bank-paths.mdc
Load: .cursor/rules/isolation_rules/Core/task-creation-notion.mdc
```

## Workflow

1. **Gather Task Info**
   - If user did not provide task title, ask for it
   - Optional: Status, Assignee, Dates/Due, Module, Man-Day, content
   - Status options: `Not Started`, `Pending`, `In Progress`, `Done`

2. **Create Task**
   - Follow `Core/task-creation-notion.mdc` (resolve project, notion-create-pages, extract TASK-ID)
   - For extra properties: use `date:Due:start`, `date:Due:is_datetime` (0), etc.
   - For relation `Project`: pass project page URL as **string** (not array)

3. **Confirm & Return TASK-ID**
   - **Return prominently**: `TASK-ID: TASK-xxxx`
   - Also return: task title, key properties, Notion link
   - Ask user if they want to update `taskId` in config – if yes, update `.cursor/notion-memory-bank.json`

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
/create-task Implement Notion sync API retry logic
```

## Next Steps

After creation, use `/van` to load the new task context, or `/plan` if planning is needed.
