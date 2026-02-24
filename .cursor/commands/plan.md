# PLAN Command - Task Planning

This command creates or updates implementation plans in the Notion Task page. It **auto-detects** whether the Plan section is empty (create full plan) or substantive (incremental update).

**CRITICAL:** PLAN phase **only** updates Notion Task page content. **Never** modify code, delete files, or change any files in the codebase. Implementation is done in `/build`.

## Plan Section Detection

**Before any write**, notion-fetch the Task page and parse content:

- **Empty/skeleton** → **Create mode**: No `# Plan` (or `## Plan`) heading, OR it exists but content between that heading and next `#` is minimal (e.g. only "Status: Pending", "Next Steps", "Run /plan", or content length < approximately 200 characters)
- **Substantive** → **Update mode**: `# Plan` or `## Plan` exists AND (has "## 1. Requirements" or "## 3. Subtasks" with actual items, or content length > approximately 200 characters)

## Memory Bank Integration (Notion)

Reads from (resolve PROJECT-/TASK- via notion-search, then notion-fetch):
- Task page (`taskId`, e.g. TASK-588) - Task requirements, complexity level, existing plan content
- activeContext page (`activeContextPageId`) - Current project context
- Project page body (`projectId`, e.g. PROJECT-123) - projectBrief (if exists)

Updates (via notion-update-page):
- Task page (`taskId`) - Adds or updates implementation plan. **Write in English** (Notion content rule).
- Project page body (`projectId`) - projectbrief (Level 2/3/4 Create mode only): When creating a full plan, after writing the Task plan, notion-update-page the Project page body with projectbrief content per workflow-level2/3/4 (Level 2: enhancement details; Level 3: feature alignment note; Level 4: architectural vision). If Project page body is empty or minimal, populate with a brief derived from the task and plan. Skip if projectbrief update is not required by the loaded workflow rules.

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
   - If `taskId` is null or empty, ask user to run `/van [description]` or `/create-task` first and stop. The Task page cannot be fetched without a valid `taskId`.

1. **Read Task Context**
   - notion-fetch Task page (resolve `taskId`) for complexity level and existing plan
   - notion-fetch activeContext page (`activeContextPageId`) for current context
   - **Determine mode**: Parse Task page content; apply Plan section detection logic → Create or Update

2. **Create Mode** (Plan empty/skeleton)
   - Create full implementation plan per complexity level
   - **Level 2:** Document planned changes, files to modify, implementation steps
   - **Level 3:** Create comprehensive plan with components, dependencies, challenges
   - **Level 4:** Create phased implementation plan with architectural considerations
   - notion-update-page with `replace_content` or `replace_content_range` to write full plan to Task page
   - Use **# Plan** as the top-level plan heading (not "# Implementation Plan: [Task name]")
   - **projectbrief (Level 2/3/4):** After writing Task plan, notion-update-page Project page body (`projectId`) with projectbrief content: Level 2 – enhancement details and scope; Level 3 – feature alignment with project goals; Level 4 – architectural vision and high-level approach. Merge with existing Project body if non-empty; do not overwrite user content entirely.

3. **Update Mode** (Plan substantive)
   - **Interpret user input**: User provides new finding, subtask to add, section to update, or refinement
   - If user did not specify what to add: ask: "What would you like to add or update? E.g. 'Avoid duplicate Active Context and Progress: search before create'"
   - **Apply incremental update** (Notion Task page only; no code/file changes):
     - Add new subtask: Find Subtasks section (e.g. `## 3. Subtasks`); use `insert_content_after` to append `- [ ] **3.X** <description>`
     - Add new section: Use `insert_content_after` with `selection_with_ellipsis` matching the last section before where to insert
     - Update existing section: Use `replace_content_range` with `selection_with_ellipsis` to replace only that section
   - **Never** use `replace_content` to overwrite the entire page in Update mode

4. **Technology Validation** (Level 2-4, Create mode only)
   - Document technology stack selection in the plan
   - Document proof-of-concept requirements and dependency checks for `/build` to execute
   - Do NOT run commands or create files—document only

5. **Identify Creative Phases** (Create mode only)
   - Flag components requiring design decisions
   - Document which components need creative exploration

6. **Update Task Page Workflow Sections**
   - Use `notion-update-page` with `command: "replace_content_range"` and `selection_with_ellipsis` matching the section.
   - Replace "## 8. Next Steps" or "## Next Steps" content **conditionally**:
     - **Level 2:** "Run `/build` for implementation."
     - **Level 3-4 (Create mode):** If creative phases were identified in Step 5 → "Run `/creative` for design exploration, or `/build` for implementation." Otherwise → "Run `/build` for implementation."
     - **Level 3-4 (Update mode):** Infer from existing plan content: if the plan has a "Creative Phases Required" (or similar) section with at least one item → "Run `/creative` for design exploration, or `/build` for implementation." Otherwise → "Run `/build` for implementation."

## Usage

```
/plan
```
Create full plan when Plan section is empty; or, if Plan exists and user provided no description, ask: "Plan already exists. Add/update something? E.g. '/plan Add subtask: handle edge case X'"

```
/plan [description of what to add]
```
- **Create mode**: Create full plan (description may inform focus)
- **Update mode**: Add or update per description (e.g. "Add subtask: handle empty taskId string")

Examples:
```
/plan Add subtask: search before create for Active Context and Progress
```

```
/plan Update section 5: add dependency on notion-search
```

## Next Steps

- **If creative phases identified:** Use `/creative` command
- **If no creative phases:** Proceed to `/build` command
