# Memory Bank Commands

This directory contains Cursor 2.0 commands that replace the deprecated custom modes feature. Each command implements a specific phase of the Memory Bank workflow with progressive rule loading to optimize context usage.

## Available Commands

### `/van` - Initialization & Entry Point
**Purpose:** Initialize Memory Bank, detect platform, determine task complexity, and route to appropriate workflows.

**When to use:** 
- Starting a new task or project
- Initializing Memory Bank structure
- Determining task complexity

**Next steps:**
- Level 1 tasks → `/build`
- Level 2-4 tasks → `/plan`

### `/plan` - Task Planning
**Purpose:** Create detailed implementation plans based on complexity level.

**When to use:**
- After `/van` determines Level 2-4 complexity
- Need to create structured implementation plan

**Next steps:**
- Creative phases identified → `/creative`
- No creative phases → `/build`

### `/plan-update` - Incremental Plan Updates
**Purpose:** Add or update plan content without replacing the existing plan. Preserves user edits and manual additions.

**When to use:**
- After `/plan` when you have new findings, subtasks, or refinements
- During planning phase when you discover issues (e.g. "重复创建了 Active Context")
- Avoid re-running `/plan` which would overwrite the entire plan

**Next steps:**
- More refinements → `/plan-update`
- Ready to implement → `/build`

### `/creative` - Design Decisions
**Purpose:** Perform structured design exploration for components requiring creative phases.

**When to use:**
- After `/plan` identifies components needing design decisions
- Need to explore architecture, UI/UX, or algorithm options

**Next steps:**
- After all creative phases complete → `/build`

### `/build` - Code Implementation
**Purpose:** Implement planned changes following the implementation plan and creative decisions.

**When to use:**
- After planning is complete (and creative phases if needed)
- Ready to start coding

**Next steps:**
- After implementation complete → `/reflect`

### `/reflect` - Task Reflection
**Purpose:** Facilitate structured reflection on completed implementation.

**When to use:**
- After `/build` completes implementation
- Need to document lessons learned and process improvements

**Next steps:**
- After reflection complete → `/archive`

### `/archive` - Task Archiving
**Purpose:** Create comprehensive archive documentation and update Memory Bank.

**When to use:**
- After `/reflect` completes reflection
- Ready to finalize task documentation

**Next steps:**
- After archiving complete → `/van` (for next task)

## Command Workflow

```
/van → /plan → /creative → /build → /reflect → /archive
  ↓       ↓        ↓         ↓         ↓          ↓
Level 1  Level   Level    Level    Level     Level
tasks    2-4     3-4      1-4      1-4       1-4
```

## Progressive Rule Loading

Each command implements progressive rule loading to optimize context usage:

1. **Core Rules** - Always loaded (main.mdc, memory-bank-paths.mdc)
2. **Mode-Specific Rules** - Loaded based on command (mode maps)
3. **Complexity-Specific Rules** - Loaded based on task complexity level
4. **Specialized Rules** - Lazy loaded only when needed (e.g., creative phase types)

This approach reduces initial token usage by ~70% compared to loading all rules at once.

## Memory Bank Integration

All commands read from and update files in the `memory-bank/` directory:

- **tasks.md** - Source of truth for task tracking
- **activeContext.md** - Current project focus
- **progress.md** - Implementation status
- **projectbrief.md** - Project foundation
- **creative/** - Creative phase documents
- **reflection/** - Reflection documents
- **archive/** - Archive documents

## Usage Examples

### Starting a New Task
```
/van Initialize project for adding user authentication feature
```

### Planning a Feature
```
/plan
```

### Exploring Design Options
```
/creative
```

### Implementing Changes
```
/build
```

### Reflecting on Completion
```
/reflect
```

### Archiving Completed Task
```
/archive
```

## Migration from Custom Modes

These commands replace the previous custom modes:
- **VAN Mode** → `/van` command
- **PLAN Mode** → `/plan` command
- **CREATIVE Mode** → `/creative` command
- **BUILD Mode** → `/build` command
- **REFLECT Mode** → `/reflect` command
- **ARCHIVE Mode** → `/archive` command

The functionality remains the same, but now uses Cursor 2.0's commands feature instead of custom modes.

