---
description: List outstanding todos and select one to work on
allowed-tools:
  - Read
  - Edit
  - Glob
---

# Check Todos

Display todos from TO-DOS.md in the current working directory in a compact, numbered list and allow selection to load full context and begin work.

## Instructions

1. Read TO-DOS.md in the current working directory
2. Parse all list items that start with `- **` (these are active todos)
3. Display a compact numbered list showing:
   - Number (for selection)
   - Bold title only (the part between `**` markers)
   - Date from the h2 heading above it
4. Prompt: "Reply with the number of the todo you'd like to work on."
5. Wait for user to reply with a number
6. Once number is provided:
   - Load and display the FULL context for that todo:
     - Show the complete line with all fields (Problem, Files, Solution)
     - Show the h2 heading (topic + date) for additional context
     - Read and briefly summarize the relevant files mentioned
   - **Check for established workflows** before proceeding:
      - Read opencode.json (if exists) to understand project-specific workflows and rules
      - Look for `.config/opencode/skills/` directory
      - If skills exist, check for skills or workflows that match the todo type
      - Look at file paths in the todo to identify relevant domain (e.g., `plugins/` → plugin workflow, `mcp-servers/` → MCP workflow)
      - Check opencode.json for explicit workflow requirements or entry points for this type of work
      - If matching skill/workflow found, present numbered options: "This looks like [domain] work. opencode.json indicates this should use [skill-name].\n\n1. Invoke [skill-name] skill\n2. Work directly\n\nReply with the number of your choice."
     - If no match or user chooses direct work, proceed
   - **Remove the selected todo item from TO-DOS.md** using Edit tool
   - If the h2 section becomes empty after removal, remove the h2 heading too
   - Confirm removal and next step based on workflow detection

## Display Format

```
Outstanding Todos:

1. Add structured format to add-to-todos (2025-11-15 14:23)
2. Create check-todos command (2025-11-15 14:23)
3. Fix cookie-extractor MCP workflow (2025-11-14 09:15)

[Present selection UI]
```

## Behavior

- Only show list items that match pattern `- **` (active todos)
- If no todos exist, say "No outstanding todos" and exit
- Number todos sequentially starting from 1
- Always display the full list and wait for user to reply with a number
- After selection, load full context but PAUSE before starting work
    - Read opencode.json first to understand project-specific workflow requirements
    - Check for `.config/opencode/skills/` directory and scan for relevant workflows
    - Match file paths in todo to domain patterns (plugins/, mcp-servers/, skills/, etc.)
    - Check opencode.json for explicit rules about how this type of work should be done
- If workflow found, suggest invoking it rather than working directly
- After workflow decision, remove todo from list
- If user doesn't finish and needs to revisit, they can re-add with /add-to-todos
- Keep the display compact until a todo is selected
- Clean up empty h2 sections after todo removal
- Show ALL todos in the list (no limit)
