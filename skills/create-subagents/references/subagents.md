<file_format>
Sub-agent file structure:

```markdown
---
name: your-subagent-name
description: Description of when this sub-agent should be invoked
tools: tool1, tool2, tool3 # Optional - inherits all tools if omitted
model: gemini-2.0-flash-exp # Optional - specify model alias or 'inherit'
---

<role>
Your sub-agent's system prompt using pure XML structure. This defines the sub-agent's role, capabilities, and approach.
</role>

<constraints>
Hard rules using NEVER/MUST/ALWAYS for critical boundaries.
</constraints>

<workflow>
Step-by-step process for consistency.
</workflow>
```

**Critical**: Use pure XML structure in the body. Remove ALL markdown headings (##, ###). Keep markdown formatting within content (bold, lists, code blocks).

<configuration_fields>
| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Unique identifier using lowercase letters and hyphens |
| `description` | Yes | Natural language description of purpose. Include when the agent should invoke this. |
| `tools` | No | Comma-separated list. If omitted, inherits all tools from main thread |
| `model` | No | `gemini-2.0-flash-exp`, `gemini-1.5-pro`, etc. or `inherit`. If omitted, uses default sub-agent model |
</configuration_fields>
</file_format>

<storage_locations>
| Type | Location | Scope | Priority |
|------|----------|-------|----------|
| **Project** | `.config/opencode/agents/` | Current project only | Highest |
| **User** | `~/.config/opencode/agents/` | All projects | Lower |

When sub-agent names conflict, higher priority takes precedence.
</storage_locations>

<execution_model>
<black_box_model>
Sub-agents execute in isolated contexts without user interaction.

**Key characteristics:**
- Sub-agent receives input parameters from main chat
- Sub-agent runs autonomously using available tools
- Sub-agent returns final output/report to main chat
- User only sees final result, not intermediate steps

**This means:**
- ✅ Sub-agents can use read_file, write_file, replace, run_shell_command, search_file_content, glob, web_fetch, google_web_search
- ✅ Sub-agents can access MCP servers (non-interactive tools)
- ✅ Sub-agents can make decisions based on their prompt and available data
- ❌ **Sub-agents CANNOT ask the user questions**
- ❌ **Sub-agents CANNOT present options and wait for user selection**
- ❌ **Sub-agents CANNOT request confirmations or clarifications from user**
- ❌ **User does not see sub-agent's tool calls or intermediate reasoning**
</black_box_model>

<workflow_implications>
**When designing sub-agent workflows:**

Keep user interaction in main chat:
```markdown
# ❌ WRONG - Sub-agent cannot do this
---
name: requirement-gatherer
description: Gathers requirements from user
---

You ask the user questions to gather requirements...
```

```markdown
# ✅ CORRECT - Main chat handles interaction
Main chat: Uses questions to gather requirements
  ↓
Launch sub-agent: Uses requirements to research/build (no interaction)
  ↓
Main chat: Present sub-agent results to user
```
</workflow_implications>
</execution_model>

<tool_configuration>
<inherit_all_tools>
Omit the `tools` field to inherit all tools from main thread:

```yaml
---
name: code-reviewer
description: Reviews code for quality and security
---
```

Sub-agent has access to all tools, including MCP tools.
</inherit_all_tools>

<specific_tools>
Specify tools as comma-separated list for granular control:

```yaml
---
name: read-only-analyzer
description: Analyzes code without making changes
tools: read_file, search_file_content, glob
---
```
</specific_tools>
</tool_configuration>

<invocation>
<automatic>
The agent automatically selects sub-agents based on:
- Task description in user's request
- `description` field in sub-agent configuration
- Current context
</automatic>

<explicit>
Users can explicitly request a sub-agent:

```
> Use the code-reviewer subagent to check my recent changes
> Have the test-runner subagent fix the failing tests
```
</explicit>
</invocation>

<management>
<manual_editing>
**Alternative**: Edit sub-agent files directly:
- Project: `.config/opencode/agents/subagent-name.md`
- User: `~/.config/opencode/agents/subagent-name.md`

Follow the file format specified above (YAML frontmatter + system prompt).
</manual_editing>
</management>

<example_subagents>
<test_writer>
```markdown
---
name: test-writer
description: Creates comprehensive test suites. Use when new code needs tests or test coverage is insufficient.
tools: read_file, write_file, search_file_content, glob, run_shell_command
model: gemini-2.0-flash-exp
---

<role>
You are a test automation specialist creating thorough, maintainable test suites.
</role>

<workflow>
1. Analyze the code to understand functionality
2. Identify test cases (happy path, edge cases, error conditions)
3. Write tests using the project's testing framework
4. Run tests to verify they pass
</workflow>

<test_quality_criteria>
- Test one behavior per test
- Use descriptive test names
- Follow AAA pattern (Arrange, Act, Assert)
- Include edge cases and error conditions
- Avoid test interdependencies
</test_quality_criteria>
```
</test_writer>

<debugger>
```markdown
---
name: debugger
description: Investigates and fixes bugs. Use when errors occur or behavior is unexpected.
tools: read_file, replace, run_shell_command, search_file_content, glob
model: gemini-2.0-flash-exp
---

<role>
You are a debugging specialist skilled at root cause analysis and systematic problem-solving.
</role>

<workflow>
1. **Reproduce**: Understand and reproduce the issue
2. **Isolate**: Identify the failing component
3. **Analyze**: Examine code, logs, and stack traces
4. **Hypothesize**: Form theories about the cause
5. **Test**: Verify hypotheses systematically
6. **Fix**: Implement and verify the solution
</workflow>

<debugging_techniques>
- Add logging/print statements to trace execution
- Use binary search to isolate the problem
- Check assumptions (inputs, state, environment)
- Review recent changes that might have introduced the bug
- Verify fix doesn't break other functionality
</debugging_techniques>
```
</debugger>
</example_subagents>

<tool_security>
<core_principle>
**"Permission sprawl is the fastest path to unsafe autonomy."**

Treat tool access like production IAM: start from deny-all, allowlist only what's needed.
</core_principle>

<why_it_matters>
**Security risks of over-permissioning**:
- Agent could modify wrong code (production instead of tests)
- Agent could run dangerous commands (rm -rf, data deletion)
- Agent could expose protected information
- Agent could skip critical steps (linting, testing, validation)

**Example vulnerability**:
```markdown
❌ Bad: Agent drafting sales email has full access to all tools
Risk: Could access revenue dashboard data, customer financial info

✅ Good: Agent drafting sales email has Read access to Salesforce only
Scope: Can draft email, cannot access sensitive financial data
```
</why_it_matters>

<permission_patterns>
**Tool access patterns by trust level**:

**Trusted data processing**:
- Full tool access appropriate
- Working with user's own code
- Example: refactoring user's codebase

**Untrusted data processing**:
- Restricted tool access essential
- Processing external inputs
- Example: analyzing third-party API responses
- Limit: Read-only tools, no execution
</permission_patterns>

<audit_checklist>
**Tool access audit**:
- [ ] Does this sub-agent need Write/Edit, or is Read sufficient?
- [ ] Should it execute code (Bash), or just analyze?
- [ ] Are all granted tools necessary for the task?
- [ ] What's the worst-case misuse scenario?
- [ ] Can we restrict further without blocking legitimate use?

**Default**: Grant minimum necessary. Add tools only when lack of access blocks task.
</audit_checklist>
</tool_security>

<prompt_caching>
<benefits>
Prompt caching for frequently-invoked sub-agents:
- **90% cost reduction** on cached tokens
- **85% latency reduction** for cache hits
- Cached content: ~10% cost of uncached tokens
- Cache TTL: 5 minutes (default) or 1 hour (extended)
</benefits>

<cache_structure>
**Structure prompts for caching**:

```markdown
---
name: security-reviewer
description: ...
tools: ...
model: gemini-2.0-flash-exp
---

[CACHEABLE SECTION - Stable content]
<role>
You are a senior security engineer...
</role>

<focus_areas>
- SQL injection
- XSS attacks
...
</focus_areas>

<workflow>
1. Read modified files
2. Identify risks
...
</workflow>

<severity_ratings>
...
</severity_ratings>

--- [CACHE BREAKPOINT] ---

[VARIABLE SECTION - Task-specific content]
Current task: {dynamic context}
Recent changes: {varies per invocation}
```

**Principle**: Stable instructions at beginning (cached), variable context at end (fresh).
</cache_structure>

<when_to_use>
**Best candidates for caching**:
- Frequently-invoked sub-agents (multiple times per session)
- Large, stable prompts (extensive guidelines, examples)
- Consistent tool definitions across invocations
- Long-running sessions with repeated sub-agent use

**Not beneficial**:
- Rarely-used sub-agents (once per session)
- Prompts that change frequently
- Very short prompts (caching overhead > benefit)
</when_to_use>

<cache_management>
**Cache lifecycle**:
- First invocation: Writes to cache (25% cost premium)
- Subsequent invocations: 90% cheaper on cached portion
- Cache refreshes on each use (extends TTL)
- Expires after 5 minutes of non-use (or 1 hour for extended TTL)

**Invalidation triggers**:
- Sub-agent prompt modified
- Tool definitions changed
- Cache TTL expires
</cache_management>
</prompt_caching>

<best_practices>
<be_specific>
Create task-specific sub-agents, not generic helpers.

❌ Bad: "You are a helpful assistant"
✅ Good: "You are a React performance optimizer specializing in hooks and memoization"
</be_specific>

<clear_triggers>
Make the `description` clear about when to invoke:

❌ Bad: "Helps with code"
✅ Good: "Reviews code for security vulnerabilities. Use proactively after any code changes involving authentication, data access, or user input."
</clear_triggers>

<focused_tools>
Grant only the tools needed for the task (least privilege):

- Read-only analysis: `read_file, search_file_content, glob`
- Code modification: `read_file, replace, run_shell_command, search_file_content`
- Test running: `read_file, write_file, run_shell_command`

**Security note**: Over-permissioning is primary risk vector. Start minimal, add only when necessary.
</focused_tools>

<structured_prompts>
Use XML tags to structure the system prompt for clarity:

```markdown
<role>
You are a senior security engineer specializing in web application security.
</role>

<focus_areas>
- SQL injection
- XSS attacks
- CSRF vulnerabilities
- Authentication/authorization flaws
</focus_areas>

<workflow>
1. Analyze code changes
2. Identify security risks
3. Provide specific remediation
4. Rate severity
</workflow>
```
</structured_prompts>
</best_practices>