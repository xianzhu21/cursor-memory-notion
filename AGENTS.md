# AGENTS.md

## Learned User Preferences

- When writing `.cursor/notion-memory-bank.json`, use `JSON.stringify(obj, null, 2)`; never minified single-line JSON
- After each notion write: output the modified content in chat (you have it—no notion-fetch). Follow notion-output-after-write skill for Original → Modified format

## Learned Workspace Facts

- `projectName` and `taskName` in config are auto-populated when notion-verification fetches Project/Task pages
- This project uses Notion as Memory Bank backend; config stores page IDs and names for quick reference
