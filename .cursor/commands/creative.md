# CREATIVE Command - Design Decisions

This command performs structured design exploration for components flagged during planning.

## Memory Bank Integration (Notion)

Reads from (resolve TASK- via notion-search, then notion-fetch):
- Task page (`taskId`, e.g. TASK-588) - Components requiring creative phases
- activeContext page (`activeContextPageId`) - Current project context

Creates (notion-create-pages):
- creative page under Task (`creativePageId` or create under resolved Task)

Updates (notion-update-page):
- Task page (`taskId`) - Records design decisions

## Progressive Rule Loading

### Step 1: Load Core Rules
```
Load: .cursor/rules/isolation_rules/main.mdc
Load: .cursor/rules/isolation_rules/Core/memory-bank-paths.mdc
Load: .cursor/rules/isolation_rules/Core/notion-retry.mdc
```

### Step 2: Load CREATIVE Mode Map
```
Load: .cursor/rules/isolation_rules/visual-maps/creative-mode-map.mdc
```

### Step 3: Load Creative Phase Enforcement
```
Load: .cursor/rules/isolation_rules/Core/creative-phase-enforcement.mdc
Load: .cursor/rules/isolation_rules/Core/creative-phase-metrics.mdc
```

### Step 4: Load Specialized Creative Rules (Lazy Loaded)
Load only when specific creative phase type is needed:

**For Architecture Design:**
```
Load: .cursor/rules/isolation_rules/Phases/CreativePhase/creative-phase-architecture.mdc
```

**For UI/UX Design:**
```
Load: .cursor/rules/isolation_rules/Phases/CreativePhase/creative-phase-uiux.mdc
```

**For Algorithm Design:**
```
Load: .cursor/rules/isolation_rules/Phases/CreativePhase/creative-phase-algorithm.mdc
```

## Workflow

1. **Verify Planning Complete**
   - notion-fetch Task page for planning completion
   - Verify creative phases are identified
   - If not complete, return to `/plan` command

2. **Identify Creative Phases**
   - Read components flagged for creative work from Task page
   - Prioritize components for design exploration

3. **Execute Creative Phase**
   For each component:
   - **🎨🎨🎨 ENTERING CREATIVE PHASE: [TYPE]**
   - Define requirements and constraints
   - Generate 2-4 design options
   - Analyze pros/cons of each option
   - Select and justify recommended approach
   - Document implementation guidelines
   - Verify solution meets requirements
   - **🎨🎨🎨 EXITING CREATIVE PHASE**

4. **Document Decisions**
   - If `creativePageId` is set: notion-fetch to verify parent = Task page. If stale (parent ≠ Task), clear in config and treat as null.
   - notion-create-pages: creative page under Task with `title: "Creative TASK-xxx"` (use taskId from config). Or use existing `creativePageId` if valid.
   - notion-update-page: Task page with design decisions

5. **Update Task Page Workflow Sections**
   - Use `notion-update-page` with `command: "replace_content_range"` and `selection_with_ellipsis` matching the section.
   - Replace "## 8. Next Steps" or "## Next Steps" content with "Run `/build` for implementation."

6. **Verify Completion**
   - Ensure all flagged components have completed creative phases
   - Mark creative phase complete in Task page

## Usage

Type `/creative` to start creative design work for components flagged in the plan.

## Next Steps

After all creative phases complete, proceed to `/build` command for implementation.

