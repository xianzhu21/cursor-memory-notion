# VAN Command - Initialization & Entry Point

This command initializes the Memory Bank system, performs platform detection, determines task complexity, and routes to appropriate workflows.

## Memory Bank Integration (Notion)

**CRITICAL:** This project uses **Notion** as Memory Bank. Use page IDs from `.cursor/notion-memory-bank.json`:
- **tasks** - Task page body (`taskId`, e.g. TASK-588 → search Tasks database)
- **activeContext** - activeContext page (`activeContextPageId`)
- **progress** - progress page (`progressPageId`)
- **projectBrief** - Project page body (`projectId`, e.g. PROJECT-123 → search Projects database)

Use `notion-search` to resolve PROJECT-/TASK- identifiers, then `notion-fetch`/`notion-update-page`/`notion-create-pages`.

## Progressive Rule Loading

This command loads rules progressively to optimize context usage:

### Step 1: Load Core Rules (Always Required)
```
Load: .cursor/rules/isolation_rules/main.mdc
Load: .cursor/rules/isolation_rules/Core/memory-bank-paths.mdc
Load: .cursor/rules/isolation_rules/Core/platform-awareness.mdc
Load: .cursor/rules/isolation_rules/Core/notion-verification.mdc
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
   - Read config: projectId, taskId, tasksDataSourceUrl, projectsDataSourceUrl
   - Treat `taskId` as "needs creation" when it is `null` or empty string `""`
   - **If taskId is null/empty**: Follow `Core/task-creation-notion.mdc`. User MUST provide task description in `/van [description]`; if missing, ask: "Please provide a task description, e.g. /van Add user authentication feature"
   - **If taskId exists AND user provided description**: notion-fetch current Task page, compare its title with user's van description
     - If description clearly differs from current task (different topic/intent): ask: "Current task (TASK-xxx: <title>) doesn't match your description. Is this a new task? Should I create it?"
     - If user confirms new task: follow task-creation-notion.mdc, update config
     - If user says no: continue with existing task

3. **Memory Bank Verification** (MANDATORY – follow notion-verification.mdc)
   - [ ] Read config: projectId, taskId, activeContextPageId, progressPageId
   - [ ] Resolve PROJECT-/TASK- via notion-search; notion-fetch Project and Task pages
   - [ ] If activeContextPageId or progressPageId is null: notion-search under Project for existing "Active Context" / "Progress"; if not found, notion-create-pages. Update config with resolved or new IDs (avoid duplicates)
   - [ ] Confirm all Project, Task, activeContext, progress pages are accessible

4. **Task Analysis**
   - notion-fetch Task page (resolve `taskId` via notion-search) for plan/checklist
   - Analyze task requirements
   - Determine complexity level (1-4)

5. **Route Based on Complexity**
   - **Level 1:** Continue in VAN mode, proceed to implementation
   - **Level 2-4:** Transition to `/plan` command

6. **Update Memory Bank**
   - notion-update-page on Task page (complexity)
   - notion-update-page on activeContext page (current focus)

## Usage

Type `/van` followed by your task description or initialization request.

Example:
```
/van Initialize project for adding user authentication feature
```

## Next Steps

- **Level 1 tasks:** Proceed directly to `/build` command
- **Level 2-4 tasks:** Use `/plan` command for detailed planning

