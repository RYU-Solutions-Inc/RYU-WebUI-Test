---
description: Switch the active project or list available projects
allowed-tools: Read, Write, Glob
argument-hint: "[project-name]"
---

Switch to project: $ARGUMENTS

This is direct Architect work — no delegation needed.

## Steps

1. If no argument provided:
   - Scan `projects/` for subdirectories
   - Read `.active-project` to identify the current active project
   - Present the list: "Available projects: {list}. Currently active: {name}."
   - Ask the analyst which project to switch to

2. If a project name was provided:
   - Validate that `projects/{name}/` exists
   - If it doesn't exist, list available projects and suggest the closest match
   - If it exists, update `.active-project` with the new project name
   - Read `projects/{name}/app-profile.md` for context
   - Present: "Switched to {name}. {brief summary from app-profile.md}"

## Output

- Updated `.active-project` file
- Inline summary of the newly active project
