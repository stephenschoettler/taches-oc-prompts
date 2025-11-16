---
description: Add todo item to TO-DOS.md with context from conversation
argument-hint: [optional todo description]
allowed-tools:
  - Read
  - Edit
  - Write
---

# Add Todo Item

Add a new todo item to TO-DOS.md in the current working directory based on the current conversation context.

## Instructions

1. Read TO-DOS.md in the current working directory (create with Write tool if it doesn't exist)
2. Check if a similar todo already exists:
   - Extract the key concept/action from the new todo
   - Search existing todos for similar titles or overlapping scope
   - If found, inform user: "A similar todo already exists: [title]. Would you like to:\n\n1. Skip adding (keep existing)\n2. Replace existing with new version\n3. Add anyway as separate item\n\nReply with the number of your choice."
   - Wait for user response before proceeding
3. Analyze the recent conversation to extract:
   - The specific problem or task discussed
   - Relevant file paths that need attention
   - Technical details (line numbers, error messages, conflicting specifications, etc.)
   - Root cause if identified
3. Create a new section at the bottom of the file with:
   - Level 2 heading (`##`) with format: `Brief Context Title - YYYY-MM-DD HH:MM`
   - One or more list items (`-`) underneath using this structured format:
     - `- **[Action verb] [Component]** - [Brief description]. **Problem:** [What's wrong/why needed]. **Files:** [Comma-separated paths with line numbers]. **Solution:** [Approach hints or constraints, if applicable].`
   - Each section should be self-contained with enough context for Claude to understand the task weeks later
   - Always include specific file paths with line numbers (e.g., `path/to/file.ts:123-145`)
   - Problem and Files fields are required; Solution field is optional
   - Use simple list items (not checkboxes) - todos are removed when work begins
4. Use the current timestamp (in format: 2025-11-15 14:23)
5. Create a brief context title (3-8 words) that describes the topic/session

## Todo Description

$ARGUMENTS

## Format Example

```markdown
## Add Todo Command Improvements - 2025-11-15 14:23

- **Add structured format to add-to-todos** - Standardize todo entries with Problem/Files/Solution pattern. **Problem:** Current todos lack consistent structure, making it hard for Claude to have enough context when revisiting tasks later. **Files:** `commands/add-to-todos.md:22-29`. **Solution:** Use inline bold labels with required Problem and Files fields, optional Solution field.

- **Create check-todos command** - Build companion command to list and select todos. **Problem:** Need workflow to review outstanding todos and load context for selected item. **Files:** `commands/check-todos.md` (new), `TO-DOS.md` (reads from). **Solution:** Parse markdown list, display numbered list, accept selection to load full context and remove item.
```

## Behavior

- If $ARGUMENTS provided: Use it as the focus/title for the todo and context title
- If no $ARGUMENTS: Infer the todo and context title from the most recent conversation topic
- Always include technical details (files, line numbers, error context)
- Append new section to the bottom of TO-DOS.md (use Edit tool, not overwrite)
- Group related todos under the same timestamp heading when appropriate
- Format timestamp using current date/time
