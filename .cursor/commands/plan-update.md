# PLAN-UPDATE Command - Incremental Plan Updates

This command **adds to or updates** the existing plan in the Task page without replacing it. Use when you have new findings, subtasks, or refinements during the planning phase.

**Difference from `/plan`**: `/plan` replaces the entire plan; `/plan-update` preserves existing content and updates incrementally.

## Memory Bank Integration (Notion)

Reads from:
- Task page (`taskId`) – existing plan content
- Config: `.cursor/notion-memory-bank.json`

Updates (via notion-update-page):
- Task page – **incremental** updates only (insert_content_after, replace_content_range)

## Rule Loading

```
Load: .cursor/rules/isolation_rules/main.mdc
Load: .cursor/rules/isolation_rules/Core/memory-bank-paths.mdc
```

## Workflow

1. **Fetch Current Plan**
   - Read config: `taskId`, `tasksDataSourceUrl`
   - **Precondition**: If `taskId` is null or empty, ask user to run `/van [description]` or `/create-task` first – plan-update requires an existing task.
   - Resolve TASK- via notion-search in Tasks database
   - **notion-fetch** Task page to get full existing content

2. **Interpret User Input**
   - User provides: new finding, subtask to add, section to update, or refinement
   - If user did not specify what to add: ask: "What would you like to add or update? E.g. 'Avoid duplicate Active Context and Progress: search before create'"

3. **Apply Incremental Update**
   - **Add new subtask**: Find the Subtasks section (e.g. `## 3. Subtasks` or `### Subtasks`); use `insert_content_after` to append `- [ ] **3.X** <description>`. **Write in English** (Notion content rule).
   - **Add new section**: Use `insert_content_after` with `selection_with_ellipsis` matching the last section before where to insert
   - **Update existing section**: Use `replace_content_range` with `selection_with_ellipsis` to replace only that section
   - **Never** use `replace_content` to overwrite the entire page

4. **Confirm**
   - Report what was added/updated
   - Note: Plan remains in Planning phase; use `/build` when ready to implement

## Usage

```
/plan-update [description of what to add]
```

Examples:
```
/plan-update Avoid duplicate Active Context and Progress: search before create
```

```
/plan-update Add subtask: handle empty taskId string
```

```
/plan-update
```
(If no description, AI will ask)

## Next Steps

- Continue planning: `/plan-update` for more refinements
- Ready to implement: `/build`
