---
description: Apply expert debugging methodology to investigate a specific issue
argument-hint: [issue description]
allowed-tools:
  - delegate_to_agent
---

<objective>
Load the debug-like-expert skill to investigate: $ARGUMENTS

This applies systematic debugging methodology with evidence gathering, hypothesis testing, and rigorous investigation.
</objective>

<process>
1. Invoke the `delegate_to_agent` tool with `agent_name="codebase_investigator"` (or appropriate debug agent if available)
2. Pass the issue description: $ARGUMENTS
3. Follow the skill's debugging methodology
4. Apply rigorous investigation and verification
</process>

<success_criteria>
- Skill successfully invoked
- Arguments passed correctly to skill
</success_criteria>