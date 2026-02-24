# BUILD Command - Code Implementation

This command implements the planned changes following the implementation plan and creative phase decisions. It enforces a test-driven approach where tests are written for all success criteria and must pass before completing each phase.

## Memory Bank Integration (Notion)

Reads from (resolve TASK- via notion-search, then notion-fetch):
- Task page (`taskId`, e.g. TASK-588) - Implementation plan and checklists
- creative page (`creativePageId`) - Design decisions (Level 3-4)
- activeContext page (`activeContextPageId`) - Current project context

Updates (notion-update-page):
- Task page (`taskId`) - Implementation progress, test results, status
- progress page (`progressPageId`) - Build status, test outcomes, observations
- **Write in English** (Notion content rule)

## Progressive Rule Loading

### Step 1: Load Core Rules
```
Load: .cursor/rules/isolation_rules/main.mdc
Load: .cursor/rules/isolation_rules/Core/memory-bank-paths.mdc
Load: .cursor/rules/isolation_rules/Core/notion-retry.mdc
Load: .cursor/rules/isolation_rules/Core/command-execution.mdc
```

### Step 2: Load BUILD Mode Map
```
Load: .cursor/rules/isolation_rules/visual-maps/build-mode-map.mdc
```

### Step 3: Load Complexity-Specific Implementation Rules
Based on complexity level from Task page:

**Level 1:**
```
Load: .cursor/rules/isolation_rules/Level1/workflow-level1.mdc
Load: .cursor/rules/isolation_rules/Level1/optimized-workflow-level1.mdc
```

**Level 2:**
```
Load: .cursor/rules/isolation_rules/Level2/workflow-level2.mdc
```

**Level 3-4:**
```
Load: .cursor/rules/isolation_rules/Level3/implementation-intermediate.mdc
Load: .cursor/rules/isolation_rules/Level4/phased-implementation.mdc
```

## Workflow

1. **Verify Prerequisites**
   - notion-fetch Task page for planning completion
   - For Level 3-4: Verify creative phase documents exist
   - Review implementation plan

2. **Determine Complexity Level**
   - Read complexity level from Task page
   - Load appropriate workflow rules

3. **Execute Implementation**

   **Level 1 (Quick Bug Fix):**
   - Review bug report
   - Examine relevant code
   - Implement targeted fix
   - Write test(s) validating the fix
   - Run tests and ensure they pass
   - notion-update-page Task page

   **Level 2 (Simple Enhancement):**
   - Review build plan
   - Examine relevant code areas
   - Implement changes sequentially
   - Write tests for each success criterion
   - Run all tests and ensure they pass
   - notion-update-page Task page

   **Level 3-4 (Feature/System):**
   - Review plan and creative decisions
   - Create directory structure
   - Build in planned phases
   - **For each phase:**
     - Write tests for all phase success criteria
     - Run tests and ensure they pass
     - Do NOT proceed to next phase until all tests pass
   - Integration testing
   - Document implementation
   - notion-update-page Task and progress pages

4. **Test-Driven Phase Completion**
   - Extract success criteria from current phase in Task page
   - Write test cases covering each success criterion
   - Execute all tests
   - **Gate:** All tests MUST pass before phase completion
   - Document test results in Task page
   - If tests fail: fix implementation, re-run tests, repeat until all pass

5. **Command Execution**
   - Document all commands executed
   - Document results and observations
   - Follow platform-specific command guidelines

6. **Verification**
   - Verify all build steps completed
   - Verify all success criteria tests pass
   - Verify changes meet requirements

7. **Update Task Page Workflow Sections**
   - Use `notion-update-page` with `command: "replace_content_range"` and `selection_with_ellipsis` matching the section.
   - If a section like "## 7. Creative Phases" or "## Creative Phases" contains "Proceed to `/build`", replace that line with "Proceeded to `/build`. Completed." (Level 1/2 may not have this section; skip if absent.)
   - Replace "## 8. Next Steps" or "## Next Steps" content with "Run `/reflect` for task review."

## Usage

Type `/build` to start implementation based on the plan in Notion Task page.

## Next Steps

After implementation complete, proceed to `/reflect` command for task review.

