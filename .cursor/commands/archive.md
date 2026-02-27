# ARCHIVE Command - Task Archiving

This command creates comprehensive archive documentation and updates the Memory Bank for future reference.

## Memory Bank Integration (Notion)

Reads from (resolve TASK- via notion-search, then notion-fetch):
- Task page (`taskId`, e.g. TASK-588) - Complete task details and checklists
- reflection page (`reflectionPageId`) - Reflection document
- progress page (`progressPageId`) - Implementation status
- creative page (`creativePageId`) - Creative phase documents (Level 3-4)

Creates (notion-create-pages):
- archive page under Task (`archivePageId` or create under resolved Task)

Updates (notion-update-page):
- Task page (`taskId`) - Mark task COMPLETE
- progress page (`progressPageId`) - Add archive reference
- activeContext page (`activeContextPageId`) - Reset for next task

## Progressive Rule Loading

### Step 1: Load Core Rules
```
Load: .cursor/rules/isolation_rules/main.mdc
Load: .cursor/rules/isolation_rules/Core/memory-bank-paths.mdc
Load: .cursor/rules/isolation_rules/Core/notion-retry.mdc
```

### Step 2: Load ARCHIVE Mode Map
```
Load: .cursor/rules/isolation_rules/visual-maps/archive-mode-map.mdc
```

### Step 3: Load Complexity-Specific Archive Rules
Based on complexity level from Task page:

**Level 1:**
```
Load: .cursor/rules/isolation_rules/Level1/quick-documentation.mdc
```

**Level 2:**
```
Load: .cursor/rules/isolation_rules/Level2/archive-basic.mdc
```

**Level 3:**
```
Load: .cursor/rules/isolation_rules/Level3/archive-intermediate.mdc
```

**Level 4:**
```
Load: .cursor/rules/isolation_rules/Level4/archive-comprehensive.mdc
```

## Workflow

1. **Verify Reflection Complete**
   - notion-fetch reflection page (`reflectionPageId`)
   - Verify reflection is complete
   - If not complete, return to `/reflect` command

2. **Create Archive Document**

   **Level 1:**
   - Create quick summary
   - Update `memory-bank/tasks.md` marking task complete

   **Level 2:**
   - Create basic archive document
   - Document changes made
   - Update `memory-bank/tasks.md` and `memory-bank/progress.md`

   **Level 3-4:**
   - Create comprehensive archive document
   - Include: Metadata, Summary, Requirements, Implementation details, Testing, Lessons Learned, References
   - Archive creative phase documents
   - Document code changes
   - Document testing approach
   - Summarize lessons learned
   - Update all Memory Bank files

3. **Archive Document Structure**
   ```
   # Task Archive: [Task Name]
   
   ## Metadata
   - Task ID, dates, complexity level
   
   ## Summary
   Brief overview of the task
   
   ## Requirements
   What the task needed to accomplish
   
   ## Implementation
   How the task was implemented
   
   ## Testing
   How the solution was verified
   
   ## Lessons Learned
   Key takeaways from the task
   
   ## References
   Use `<mention-page url="[URL]">[Title]</mention-page>` for page links (NOT `<page url="...">` which moves pages)
   ```

4. **Update Memory Bank**
   - If `archivePageId` is set: notion-fetch to verify parent = Task page. If stale (parent ≠ Task), clear in config and treat as null.
   - notion-create-pages: archive page under Task with `title: "Archive TASK-xxx"` (use taskId from config). Or use existing `archivePageId` if valid.
   - **MANDATORY** notion-update-page Task page properties (do NOT skip):
     - Status: "Done"
     - **Dates**: notion-fetch Task first. If `date:Dates:start` exists → keep it, set `date:Dates:end` = today. If empty → `date:Dates:start` = today, `date:Dates:end` = today. Run `date +%Y-%m-%d` for today. Use `update_properties` with `date:Dates:start`, `date:Dates:end`, `date:Dates:is_datetime` (0).
   - Use `replace_content_range` to replace "## 8. Next Steps" or "## Next Steps" content with "Run `/van` for next task."
   - Use `replace_content_range` to replace "# Reflection" with "# Reflection & Archive" (heading only; keep child pages using `<page url="...">` – required to preserve structure)
   - notion-update-page progress page: add archive reference using `<mention-page url="[archivePageUrl]">Archive TASK-xxx</mention-page>`
   - notion-update-page activeContext page: reset for next task
   - Clear completed task details from Task page (keep structure)

## Usage

Type `/archive` to archive the completed task after reflection is done.

## Next Steps

After archiving complete, use `/van` command to start the next task.

