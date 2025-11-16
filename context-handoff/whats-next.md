---
name: whats-next
description: Analyze the current conversation and create a handoff document for continuing this work in a fresh context
allowed-tools: [Bash, Read, Edit, Write, web_search, web_fetch]
---

Analyze the current conversation and create a handoff document for continuing this work in a fresh context.

<instructions>
1. Identify the **original task** from this conversation - what was initially requested
2. Determine what has been **completed** toward that original task
3. Determine what **remains** to complete that same task (do NOT add new scope)
4. Note any **important context** (decisions made, approaches chosen, blockers encountered)
5. Write this information to `whats-next.md` in the current working directory using the XML structure shown below
</instructions>

<analysis_framework>
Use this structure to analyze the conversation before writing the file:

<original_task>
What was the user's initial request? What specific task were we trying to complete?
</original_task>

<work_completed>
What has been done toward completing that original task?

- Files modified
- Changes made
- Problems solved
  </work_completed>

<work_remaining>
What still needs to be done to complete the ORIGINAL task (not new tasks)?

- Specific steps remaining
- Files still to modify
- Known issues to resolve
  </work_remaining>

<context_decisions>
What context is essential to continue effectively?

- Technical decisions made
- Approaches chosen (and why)
- Blockers or constraints
- Any "gotchas" discovered
  </context_decisions>
  </analysis_framework>

<output_format>
Write `whats-next.md` with this exact structure:

```xml
<original_task>
[The specific task that was requested]
</original_task>

<work_completed>
[What has been accomplished toward completing that task]
</work_completed>

<work_remaining>
[What still needs to be done to finish the original task]
</work_remaining>

<context>
[Essential context, decisions, constraints, or gotchas]
</context>
```

</output_format>

<important>
- Focus ONLY on completing the original task - do not add new scope or suggest next steps beyond what was asked
- Be specific about file paths, line numbers, function names where relevant
- If the original task is complete, state that clearly in work_remaining
- The goal is seamless continuation, not planning new work
</important>

After writing the file, confirm:
"✓ Created whats-next.md - reference with @whats-next.md in a new chat to continue"
