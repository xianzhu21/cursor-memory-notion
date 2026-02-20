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
**Purpose:** Create or update implementation plans in the Notion Task page. Auto-detects: if Plan section is empty → creates full plan; if Plan exists → updates incrementally.

**When to use:**
- After `/van` determines Level 2-4 complexity
- Need to create structured implementation plan
- Need to add subtasks or refinements (use `/plan [description]`)

**Next steps:**
- Creative phases identified → `/creative`
- No creative phases → `/build`

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

## Phase Scope

| Command | Scope | Does NOT |
|---------|-------|----------|
| `/van` | Init, task creation, complexity | Implement (Level 2-4) |
| `/plan` | Notion Task page only | Modify code or files |
| `/creative` | Design decisions (Notion) | Implement |
| `/build` | Code implementation | Plan or design |
| `/reflect` | Reflection (Notion) | Implement |
| `/archive` | Archive (Notion) | Implement |

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

**This fork uses Notion** as the Memory Bank backend (not local files). All commands read from and update Notion pages:

- **Task page** (`taskId`) - Source of truth for task tracking, plan, checklists
- **activeContext page** - Current project focus
- **progress page** - Implementation status
- **Project page** (`projectId`) - projectBrief
- **Creative / Reflection / Archive** - Subpages under Task when applicable

See `.cursor/notion-memory-bank.json` and `memory-bank-paths.mdc` for configuration.

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

