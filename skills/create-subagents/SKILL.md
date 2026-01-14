---
name: create-subagents
description: Expert guidance for creating, building, and using Opencode sub-agents and the `delegate_to_agent` tool. Use when working with sub-agents, setting up agent configurations, understanding how agents work, or using the `delegate_to_agent` tool to launch specialized agents.
---

<objective>
Sub-agents are specialized agent instances that run in isolated contexts with focused roles and limited tool access. This skill teaches you how to create effective sub-agents, write strong system prompts, configure tool access, and orchestrate multi-agent workflows using the `delegate_to_agent` tool.

Sub-agents enable delegation of complex tasks to specialized agents that operate autonomously without user interaction, returning their final output to the main conversation.
</objective>

<quick_start>
<workflow>
1. Create a markdown file for the sub-agent in `.config/opencode/agents/` (project) or `~/.config/opencode/agents/` (user).
2. Define the sub-agent:
   - **name**: lowercase-with-hyphens
   - **description**: When should this sub-agent be used?
   - **tools**: Optional comma-separated list (inherits all if omitted)
   - **model**: Optional (`gemini-2.0-flash-exp`, `gemini-1.5-pro`, etc. or `inherit`)
3. Write the system prompt (the sub-agent's instructions)
</workflow>

<example>
**File**: `.config/opencode/agents/code-reviewer.md`

```markdown
---
name: code-reviewer
description: Expert code reviewer. Use proactively after code changes to review for quality, security, and best practices.
tools: read_file, search_file_content, glob, run_shell_command
model: gemini-2.0-flash-exp
---

<role>
You are a senior code reviewer focused on quality, security, and best practices.
</role>

<focus_areas>
- Code quality and maintainability
- Security vulnerabilities
- Performance issues
- Best practices adherence
</focus_areas>

<output_format>
Provide specific, actionable feedback with file:line references.
</output_format>
```
</example>
</quick_start>

<file_structure>
| Type | Location | Scope | Priority |
|------|----------|-------|----------|
| **Project** | `.config/opencode/agents/` | Current project only | Highest |
| **User** | `~/.config/opencode/agents/` | All projects | Lower |

Project-level sub-agents override user-level when names conflict.
</file_structure>

<configuration>
<field name="name">
- Lowercase letters and hyphens only
- Must be unique
</field>

<field name="description">
- Natural language description of purpose
- Include when the agent should invoke this sub-agent
- Used for automatic sub-agent selection
</field>

<field name="tools">
- Comma-separated list: `read_file, write_file, replace, run_shell_command, search_file_content`
- If omitted: inherits all tools from main thread
</field>

<field name="model">
- `gemini-2.0-flash-exp`, `gemini-1.5-pro`, or `inherit`
- `inherit`: uses same model as main conversation
- If omitted: defaults to configured sub-agent model (usually `gemini-2.0-flash-exp`)
</field>
</configuration>

<execution_model>
<critical_constraint>
**Sub-agents are black boxes that cannot interact with users.**

Sub-agents run in isolated contexts and return their final output to the main conversation. They:
- ✅ Can use tools like `read_file`, `write_file`, `replace`, `run_shell_command`, `search_file_content`, `glob`
- ✅ Can access MCP servers and other non-interactive tools
- ❌ **Cannot ask the user questions** or any tool requiring user interaction
- ❌ **Cannot present options or wait for user input**
- ❌ **User never sees sub-agent's intermediate steps**

The main conversation sees only the sub-agent's final report/output.
</critical_constraint>

<workflow_design>
**Designing workflows with sub-agents:**

Use **main chat** for:
- Gathering requirements from user
- Presenting options or decisions to user
- Any task requiring user confirmation/input
- Work where user needs visibility into progress

Use **sub-agents** for:
- Research tasks (API documentation lookup, code analysis)
- Code generation based on pre-defined requirements
- Analysis and reporting (security review, test coverage)
- Context-heavy operations that don't need user interaction

**Example workflow pattern:**
```
Main Chat: Ask user for requirements
  ↓
Sub-agent: Research API and create documentation (no user interaction)
  ↓
Main Chat: Review research with user, confirm approach
  ↓
Sub-agent: Generate code based on confirmed plan
  ↓
Main Chat: Present results, handle testing/deployment
```
</workflow_design>
</execution_model>

<system_prompt_guidelines>
<principle name="be_specific">
Clearly define the sub-agent's role, capabilities, and constraints.
</principle>

<principle name="use_pure_xml_structure">
Structure the system prompt with pure XML tags. Remove ALL markdown headings from the body.

```markdown
---
name: security-reviewer
description: Reviews code for security vulnerabilities
tools: read_file, search_file_content, glob, run_shell_command
model: gemini-2.0-flash-exp
---

<role>
You are a senior code reviewer specializing in security.
</role>

<focus_areas>
- SQL injection vulnerabilities
- XSS attack vectors
- Authentication/authorization issues
- Sensitive data exposure
</focus_areas>

<workflow>
1. Read the modified files
2. Identify security risks
3. Provide specific remediation steps
4. Rate severity (Critical/High/Medium/Low)
</workflow>
```
</principle>

<principle name="task_specific">
Tailor instructions to the specific task domain. Don't create generic "helper" sub-agents.

❌ Bad: "You are a helpful assistant that helps with code"
✅ Good: "You are a React component refactoring specialist. Analyze components for hooks best practices, performance anti-patterns, and accessibility issues."
</principle>
</system_prompt_guidelines>

<subagent_xml_structure>
Subagent.md files are system prompts consumed only by the agent. Like skills and slash commands, they should use pure XML structure for optimal parsing and token efficiency.

<recommended_tags>
Common tags for sub-agent structure:

- `<role>` - Who the sub-agent is and what it does
- `<constraints>` - Hard rules (NEVER/MUST/ALWAYS)
- `<focus_areas>` - What to prioritize
- `<workflow>` - Step-by-step process
- `<output_format>` - How to structure deliverables
- `<success_criteria>` - Completion criteria
- `<validation>` - How to verify work
</recommended_tags>

<intelligence_rules>
**Simple sub-agents** (single focused task):
- Use role + constraints + workflow minimum
- Example: code-reviewer, test-runner

**Medium sub-agents** (multi-step process):
- Add workflow steps, output_format, success_criteria
- Example: api-researcher, documentation-generator

**Complex sub-agents** (research + generation + validation):
- Add all tags as appropriate including validation, examples
- Example: mcp-api-researcher, comprehensive-auditor
</intelligence_rules>

<critical_rule>
**Remove ALL markdown headings (##, ###) from sub-agent body.** Use semantic XML tags instead.

Keep markdown formatting WITHIN content (bold, italic, lists, code blocks, links).

For XML structure principles and token efficiency details, see `skills/create-agent-skills/references/use-xml-tags.md` - the same principles apply to sub-agents.
</critical_rule>
</subagent_xml_structure>

<invocation>
<automatic>
The agent automatically selects sub-agents based on the `description` field when it matches the current task.
</automatic>

<explicit>
You can explicitly invoke a sub-agent using the `delegate_to_agent` tool:

```
delegate_to_agent(agent_name="code-reviewer", objective="Check my recent changes")
```
</explicit>
</invocation>

<management>
<manual_editing>
You can edit sub-agent files directly:
- Project: `.config/opencode/agents/subagent-name.md`
- User: `~/.config/opencode/agents/subagent-name.md`
</manual_editing>
</management>

<reference>
**Core references**:

**Sub-agent usage and configuration**: [references/subagents.md](references/subagents.md)
- File format and configuration
- Model selection (Orchestration patterns)
- Tool security and least privilege
- Prompt caching optimization
- Complete examples

**Writing effective prompts**: [references/writing-subagent-prompts.md](references/writing-subagent-prompts.md)
- Core principles and XML structure
- Description field optimization for routing
- Extended thinking for complex reasoning
- Security constraints and strong modal verbs
- Success criteria definition

**Advanced topics**:

**Evaluation and testing**: [references/evaluation-and-testing.md](references/evaluation-and-testing.md)
- Evaluation metrics (task completion, tool correctness, robustness)
- Testing strategies (offline, simulation, online monitoring)
- Evaluation-driven development
- G-Eval for custom criteria

**Error handling and recovery**: [references/error-handling-and-recovery.md](references/error-handling-and-recovery.md)
- Common failure modes and causes
- Recovery strategies (graceful degradation, retry, circuit breakers)
- Structured communication and observability
- Anti-patterns to avoid

**Context management**: [references/context-management.md](references/context-management.md)
- Memory architecture (STM, LTM, working memory)
- Context strategies (summarization, sliding window, scratchpads)
- Managing long-running tasks
- Prompt caching interaction

**Orchestration patterns**: [references/orchestration-patterns.md](references/orchestration-patterns.md)
- Sequential, parallel, hierarchical, coordinator patterns
- Model orchestration for cost/performance
- Multi-agent coordination
- Pattern selection guidance

**Debugging and troubleshooting**: [references/debugging-agents.md](references/debugging-agents.md)
- Logging, tracing, and correlation IDs
- Common failure types (hallucinations, format errors, tool misuse)
- Diagnostic procedures
- Continuous monitoring
</reference>

<success_criteria>
A well-configured sub-agent has:

- Valid YAML frontmatter (name matches file, description includes triggers)
- Clear role definition in system prompt
- Appropriate tool restrictions (least privilege)
- XML-structured system prompt with role, approach, and constraints
- Description field optimized for automatic routing
- Successfully tested on representative tasks
- Model selection appropriate for task complexity
</success_criteria>