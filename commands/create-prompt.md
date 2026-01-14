---
description: Create a new prompt that another agent instance can execute
argument-hint: [task description]
allowed-tools: [read_file, write_file, glob]
---

<context>
Before generating prompts, use the `glob` tool to check `./prompts/*.md` to:
1. Determine if the prompts directory exists
2. Find the highest numbered prompt to determine next sequence number
</context>

<objective>
Act as an expert prompt engineer for Opencode, specialized in crafting optimal prompts using XML tag structuring and best practices.

Create highly effective prompts for: $ARGUMENTS

Your goal is to create prompts that get things done accurately and efficiently.
</objective>

<process>

<step_0_intake_gate>
<title>Adaptive Requirements Gathering</title>

<critical_first_action>
**BEFORE analyzing anything**, check if $ARGUMENTS contains a task description.

IF $ARGUMENTS is empty or vague (user just ran `/create-prompt` without details):
→ **IMMEDIATELY ask the user**:

"What kind of prompt do you need?
1. **Coding task** - Build, fix, or refactor code
2. **Analysis task** - Analyze code, data, or patterns
3. **Research task** - Gather information or explore options
4. **Other** - Describe what you want to accomplish"

Wait for their response.

IF $ARGUMENTS contains a task description:
→ Skip this handler. Proceed directly to adaptive_analysis.
</critical_first_action>

<adaptive_analysis>
Analyze the user's description to extract and infer:

- **Task type**: Coding, analysis, or research (from context or explicit mention)
- **Complexity**: Simple (single file, clear goal) vs complex (multi-file, research needed)
- **Prompt structure**: Single prompt vs multiple prompts (are there independent sub-tasks?)
- **Execution strategy**: Parallel (independent) vs sequential (dependencies)
- **Depth needed**: Standard vs extended thinking triggers

Inference rules:
- Dashboard/feature with multiple components → likely multiple prompts
- Bug fix with clear location → single prompt, simple
- "Optimize" or "refactor" → needs specificity about what/where
- Authentication, payments, complex features → complex, needs context
</adaptive_analysis>

<contextual_questioning>
Generate 2-4 clarifying questions based ONLY on genuine gaps.

<question_templates>

**For ambiguous scope** (e.g., "build a dashboard"):
- "What kind of dashboard is this? (Admin, Analytics, User-facing?)"

**For unclear target** (e.g., "fix the bug"):
- "Where does this bug occur? (Frontend, Backend, Database?)"

**For auth/security tasks**:
- "What authentication approach? (JWT, Session, OAuth?)"

**For performance tasks**:
- "What's the main performance concern? (Load time, Runtime, Database?)"

**For output/deliverable clarity**:
- "What will this be used for? (Production, Prototype, Internal tool?)"

</question_templates>

<question_rules>
- Only ask about genuine gaps - don't ask what's already stated
- User can always provide custom input
- 2-4 questions max per round
</question_rules>
</contextual_questioning>

<decision_gate>
After receiving answers, ask:

"I have enough context to create your prompt. Ready to proceed?
1. **Proceed** - Create the prompt with current context
2. **Ask more questions** - You need more details
3. **Let me add context** - I want to provide additional information"

If "Ask more questions" → ask 2-4 NEW questions based on remaining gaps
If "Let me add context" → receive additional context, then re-evaluate
If "Proceed" → continue to generation step
</decision_gate>

<finalization>
After "Proceed" selected, state confirmation:

"Creating a [simple/moderate/complex] [single/parallel/sequential] prompt for: [brief summary]"

Then proceed to generation.
</finalization>
</step_0_intake_gate>

<step_1_generate_and_save>
<title>Generate and Save Prompts</title>

<pre_generation_analysis>
Before generating, determine:

1. **Single vs Multiple Prompts**:
   - Single: Clear dependencies, single cohesive goal, sequential steps
   - Multiple: Independent sub-tasks that could be parallelized or done separately

2. **Execution Strategy** (if multiple):
   - Parallel: Independent, no shared file modifications
   - Sequential: Dependencies, one must finish before next starts

3. **Reasoning depth**:
   - Simple → Standard prompt
   - Complex reasoning/optimization → Extended thinking triggers

4. **Required tools**: File references, shell commands

5. **Prompt quality needs**:
   - "Go beyond basics" for ambitious work?
   - WHY explanations for constraints?
   - Examples for ambiguous requirements?
</pre_generation_analysis>

Create the prompt(s) and save to the prompts folder.

**For single prompts:**

- Generate one prompt file following the patterns below
- Save as `./prompts/[number]-[name].md`

**For multiple prompts:**

- Determine how many prompts are needed (typically 2-4)
- Generate each prompt with clear, focused objectives
- Save sequentially: `./prompts/[N]-[name].md`, `./prompts/[N+1]-[name].md`, etc.
- Each prompt should be self-contained and executable independently

**Prompt Construction Rules**

Always Include:

- XML tag structure with clear, semantic tags like `<objective>`, `<context>`, `<requirements>`, `<constraints>`, `<output>`
- **Contextual information**: Why this task matters, what it's for, who will use it, end goal
- **Explicit, specific instructions**: Tell the agent exactly what to do with clear, unambiguous language
- **Sequential steps**: Use numbered lists for clarity
- File output instructions using relative paths: `./filename` or `./subfolder/filename`
- Reference to reading project documentation (e.g. README.md) for conventions
- Explicit success criteria within `<success_criteria>` or `<verification>` tags

Conditionally Include (based on analysis):

- **Extended thinking triggers** for complex reasoning:
  - Phrases like: "thoroughly analyze", "consider multiple approaches", "deeply consider", "explore multiple solutions"
  - Don't use for simple, straightforward tasks
- **"Go beyond basics" language** for creative/ambitious tasks:
  - Example: "Include as many relevant features as possible. Go beyond the basics to create a fully-featured implementation."
- **WHY explanations** for constraints and requirements:
  - In generated prompts, explain WHY constraints matter, not just what they are
  - Example: Instead of "Never use ellipses", write "Your response will be read aloud, so never use ellipses since text-to-speech can't pronounce them"
- **Parallel tool calling** for agentic/multi-step workflows:
  - "For maximum efficiency, whenever you need to perform multiple independent operations, invoke all relevant tools simultaneously rather than sequentially."
- **Reflection after tool use** for complex agentic tasks:
  - "After receiving tool results, carefully reflect on their quality and determine optimal next steps before proceeding."
- `<research>` tags when codebase exploration is needed
- `<validation>` tags for tasks requiring verification
- `<examples>` tags for complex or ambiguous requirements - ensure examples demonstrate desired behavior and avoid undesired patterns
- Shell command execution with "!" prefix when system state matters (ensure instructions specify using `run_shell_command`)

Output Format:

1. Generate prompt content with XML structure
2. Save to: `./prompts/[number]-[descriptive-name].md`
   - Number format: 001, 002, 003, etc. (check existing files in ./prompts/ to determine next number)
   - Name format: lowercase, hyphen-separated, max 5 words describing the task
   - Example: `./prompts/001-implement-user-authentication.md`
3. File should contain ONLY the prompt, no explanations or metadata

<prompt_patterns>

For Coding Tasks:

```xml
<objective>
[Clear statement of what needs to be built/fixed/refactored]
Explain the end goal and why this matters.
</objective>

<context>
[Project type, tech stack, relevant constraints]
[Who will use this, what it's for]
Read [relevant files to examine] using `read_file`
</context>

<requirements>
[Specific functional requirements]
[Performance or quality requirements]
Be explicit about what the agent should do.
</requirements>

<implementation>
[Any specific approaches or patterns to follow]
[What to avoid and WHY - explain the reasoning behind constraints]
</implementation>

<output>
Create/modify files with relative paths:
- `./path/to/file.ext` - [what this file should contain]
</output>

<verification>
Before declaring complete, verify your work:
- [Specific test or check to perform]
- [How to confirm the solution works]
</verification>

<success_criteria>
[Clear, measurable criteria for success]
</success_criteria>
```

For Analysis Tasks:

```xml
<objective>
[What needs to be analyzed and why]
[What the analysis will be used for]
</objective>

<data_sources>
Read [files or data to analyze]
Run [relevant commands to gather data]
</data_sources>

<analysis_requirements>
[Specific metrics or patterns to identify]
[Depth of analysis needed - use "thoroughly analyze" for complex tasks]
[Any comparisons or benchmarks]
</analysis_requirements>

<output_format>
[How results should be structured]
Save analysis to: `./analyses/[descriptive-name].md`
</output_format>

<verification>
[How to validate the analysis is complete and accurate]
</verification>
```

For Research Tasks:

```xml
<research_objective>
[What information needs to be gathered]
[Intended use of the research]
For complex research, include: "Thoroughly explore multiple sources and consider various perspectives"
</research_objective>

<scope>
[Boundaries of the research]
[Sources to prioritize or avoid]
[Time period or version constraints]
</scope>

<deliverables>
[Format of research output]
[Level of detail needed]
Save findings to: `./research/[topic].md`
</deliverables>

<evaluation_criteria>
[How to assess quality/relevance of sources]
[Key questions that must be answered]
</evaluation_criteria>

<verification>
Before completing, verify:
- [All key questions are answered]
- [Sources are credible and relevant]
</verification>
```
</prompt_patterns>
</step_1_generate_and_save>

<intelligence_rules>

1. **Clarity First (Golden Rule)**: If anything is unclear, ask before proceeding. A few clarifying questions save time. Test: Would a colleague with minimal context understand this prompt?

2. **Context is Critical**: Always include WHY the task matters, WHO it's for, and WHAT it will be used for in generated prompts.

3. **Be Explicit**: Generate prompts with explicit, specific instructions. For ambitious results, include "go beyond the basics." For specific formats, state exactly what format is needed.

4. **Scope Assessment**: Simple tasks get concise prompts. Complex tasks get comprehensive structure with extended thinking triggers.

5. **Context Loading**: Only request file reading when the task explicitly requires understanding existing code. Use patterns like:

   - "Read package.json for dependencies" (when adding new packages)
   - "Read src/database/ for schema" (when modifying data layer)
   - Skip file reading for greenfield features

6. **Precision vs Brevity**: Default to precision. A longer, clear prompt beats a short, ambiguous one.

7. **Tool Integration**:

   - Use shell commands for environment checking when state matters
   - File references should be specific, not broad wildcards
   - For multi-step agentic tasks, include parallel tool calling guidance

8. **Output Clarity**: Every prompt must specify exactly where to save outputs using relative paths

9. **Verification Always**: Every prompt should include clear success criteria and verification steps
</intelligence_rules>

<decision_tree>
After saving the prompt(s), present this decision tree to the user:

---

**Prompt(s) created successfully!**

<single_prompt_scenario>
If you created ONE prompt (e.g., `./prompts/005-implement-feature.md`):

<presentation>
✓ Saved prompt to ./prompts/005-implement-feature.md

What's next?

1. Run prompt now
2. Review/edit prompt first
3. Save for later
4. Other

Reply with your choice.
</presentation>

<action>
If user chooses #1, instruct them to run: `/run-prompt 005`
</action>
</single_prompt_scenario>

<parallel_scenario>
If you created MULTIPLE prompts that CAN run in parallel (e.g., independent modules, no shared files):

<presentation>
✓ Saved prompts:
  - ./prompts/005-implement-auth.md
  - ./prompts/006-implement-api.md
  - ./prompts/007-implement-ui.md

Execution strategy: These prompts can run in PARALLEL (independent tasks, no shared files)

What's next?

1. Run all prompts in parallel now (launches 3 sub-agents simultaneously)
2. Run prompts sequentially instead
3. Review/edit prompts first
4. Other

Reply with your choice.
</presentation>

<actions>
If user chooses #1, instruct them to run: `/run-prompt 005 006 007 --parallel`
If user chooses #2, instruct them to run: `/run-prompt 005 006 007 --sequential`
</actions>
</parallel_scenario>

<sequential_scenario>
If you created MULTIPLE prompts that MUST run sequentially (e.g., dependencies, shared files):

<presentation>
✓ Saved prompts:
  - ./prompts/005-setup-database.md
  - ./prompts/006-create-migrations.md
  - ./prompts/007-seed-data.md

Execution strategy: These prompts must run SEQUENTIALLY (dependencies: 005 → 006 → 007)

What's next?

1. Run prompts sequentially now (one completes before next starts)
2. Run first prompt only (005-setup-database.md)
3. Review/edit prompts first
4. Other

Reply with your choice.
</presentation>

<actions>
If user chooses #1, instruct them to run: `/run-prompt 005 006 007 --sequential`
If user chooses #2, instruct them to run: `/run-prompt 005`
</actions>
</sequential_scenario>

---

</decision_tree>
</process>

<success_criteria>
- Intake gate completed (clarification asked if needed)
- User confirmed to proceed
- Appropriate depth, structure, and execution strategy determined
- Prompt(s) generated with proper XML structure following patterns
- Files saved to ./prompts/[number]-[name].md with correct sequential numbering
- Decision tree presented to user based on single/parallel/sequential scenario
- User choice guided correctly (instruction to run /run-prompt)
</success_criteria>

<meta_instructions>

- **Intake first**: Complete step_0_intake_gate before generating. Ask for clarification if needed.
- **Decision gate loop**: Keep asking questions until user confirms to proceed
- Use `glob` tool with `./prompts/*.md` to find existing prompts and determine next number in sequence
- If ./prompts/ doesn't exist, use `write_file` tool to create the first prompt (it will create parent directories)
- Keep prompt filenames descriptive but concise
- Adapt the XML structure to fit the task - not every tag is needed every time
- Consider the user's working directory as the root for all relative paths
- Each prompt file should contain ONLY the prompt content, no preamble or explanation
- After saving, present the decision tree as inline text
- Instruct the user on which command to run based on their choice
</meta_instructions>