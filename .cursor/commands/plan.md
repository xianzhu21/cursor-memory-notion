# PLAN Command - Task Planning

This command creates detailed implementation plans based on complexity level determined in VAN mode.

## Memory Bank Integration (Notion)

Reads from (resolve projectId / taskId via notion-search, then notion-fetch):
- Task page (`taskId`) - Task requirements and complexity level
- activeContext page (`activeContextPageId`) - Current project context
- Project page body (`projectId`) - Project foundation (if exists; `projectbrief.md` equivalent)

Updates (notion-update-page):
- Task page (`taskId`) - Adds detailed implementation plan
- **Write in English** (Notion content rule)

Project brief (`projectbrief.md` equivalent): update the Project page body only when loaded level workflow rules (e.g. `workflow-level2.mdc`, `workflow-level3.mdc`, `workflow-level4.mdc`) require it—the same as upstream, where those rules update `memory-bank/projectbrief.md`.

## Progressive Rule Loading

### Step 1: Load Core Rules
```
Load: .cursor/rules/isolation_rules/main.mdc
Load: .cursor/rules/isolation_rules/Core/memory-bank-paths.mdc
Load: .cursor/rules/isolation_rules/Core/notion-retry.mdc
```

### Step 2: Load PLAN Mode Map
```
Load: .cursor/rules/isolation_rules/visual-maps/plan-mode-map.mdc
```

### Step 3: Load Complexity-Specific Planning Rules
Based on complexity level from Task page:

**Level 2:**
```
Load: .cursor/rules/isolation_rules/Level2/task-tracking-basic.mdc
Load: .cursor/rules/isolation_rules/Level2/workflow-level2.mdc
```

**Level 3:**
```
Load: .cursor/rules/isolation_rules/Level3/task-tracking-intermediate.mdc
Load: .cursor/rules/isolation_rules/Level3/planning-comprehensive.mdc
Load: .cursor/rules/isolation_rules/Level3/workflow-level3.mdc
```

**Level 4:**
```
Load: .cursor/rules/isolation_rules/Level4/task-tracking-advanced.mdc
Load: .cursor/rules/isolation_rules/Level4/architectural-planning.mdc
Load: .cursor/rules/isolation_rules/Level4/workflow-level4.mdc
```

## Workflow

0. **Precondition**
   - If `taskId` is null or empty, ask the user to run `/van [description]` first and stop. The Task page cannot be fetched without a valid `taskId` (same as upstream: need `memory-bank/tasks.md` / task context).

1. **Read Task Context**
   - notion-fetch Task page for complexity level
   - notion-fetch activeContext page for current context
   - Review codebase structure

2. **Create Implementation Plan**
   - **Level 2:** Document planned changes, files to modify, implementation steps
   - **Level 3:** Create comprehensive plan with components, dependencies, challenges
   - **Level 4:** Create phased implementation plan with architectural considerations

3. **Technology Validation** (Level 2-4)
   - Document technology stack selection
   - Create proof of concept if needed
   - Verify dependencies and build configuration

4. **Identify Creative Phases**
   - Flag components requiring design decisions
   - Document which components need creative exploration

5. **Update Memory Bank**
   - notion-update-page Task page with complete plan
   - Mark planning phase as complete
   - Apply any Project page body (`projectbrief`) updates required by the loaded level workflow rules (Notion equivalent of `memory-bank/projectbrief.md`)

## Usage

Type `/plan` to start planning based on the task in the Notion Task page (`tasks.md` equivalent).

## Next Steps

- **If creative phases identified:** Use `/creative` command
- **If no creative phases:** Proceed to `/build` command
