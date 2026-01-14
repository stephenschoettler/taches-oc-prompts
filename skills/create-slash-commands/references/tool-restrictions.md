# Tool Restrictions Reference

Official documentation on restricting tool access in slash commands.

## Why Restrict Tools

Tool restrictions provide:
- **Security**: Prevent accidental destructive operations
- **Focus**: Limit scope for specialized commands
- **Safety**: Ensure commands only perform intended operations

## allowed-tools Field

**Location**: YAML frontmatter

**Format**: Array of tool names or patterns

**Default**: If omitted, all tools available

## Basic Patterns

### Array Format

```yaml
---
description: My command
allowed-tools: [read_file, replace, write_file]
---
```

### Single Tool

```yaml
---
description: Thinking command
allowed-tools: think
---
```

## Bash Command Restrictions

**Source**: Official Opencode documentation

Restrict bash commands to specific patterns using wildcards.

### Git-Only Commands

```yaml
---
description: Create a git commit
allowed-tools: run_shell_command(git add:*), run_shell_command(git status:*), run_shell_command(git commit:*)
---
```

**Allows**:
- `git add <anything>`
- `git status <anything>`
- `git commit <anything>`

**Prevents**:
- `rm -rf`
- `curl <url>`
- Any non-git bash commands

### NPM Script Restrictions

```yaml
---
description: Run tests and lint
allowed-tools: run_shell_command(npm test:*), run_shell_command(npm run lint:*)
---
```

**Allows**:
- `npm test`
- `npm test -- --watch`
- `npm run lint`
- `npm run lint:fix`

**Prevents**:
- `npm install malicious-package`
- `npm run deploy`
- Other npm commands

### Multiple Bash Patterns

```yaml
---
description: Development workflow
allowed-tools: run_shell_command(git status:*), run_shell_command(npm test:*), run_shell_command(npm run build:*)
---
```

Combines multiple bash command patterns.

## Common Tool Restriction Patterns

### Pattern 1: Git Workflows

**Use case**: Commands that create commits, check status, etc.

```yaml
---
description: Create a git commit
allowed-tools: run_shell_command(git add:*), run_shell_command(git status:*), run_shell_command(git diff:*), run_shell_command(git commit:*)
---

Current status: ! `git status`
Changes: ! `git diff HEAD`

Create a commit for these changes.
```

**Security benefit**: Cannot accidentally run destructive commands like `rm -rf` or `curl malicious-site.com`

### Pattern 2: Read-Only Analysis

**Use case**: Commands that analyze code without modifying it

```yaml
---
description: Analyze codebase for pattern
allowed-tools: [read_file, search_file_content, glob]
---

Search codebase for: $ARGUMENTS
```

**Security benefit**: Cannot write files or execute code

### Pattern 3: Thinking-Only Commands

**Use case**: Deep analysis or planning without file operations

```yaml
---
description: Analyze problem from first principles
allowed-tools: think
---

Analyze the current problem from first principles.
```

**Focus benefit**: The agent focuses purely on reasoning, no file operations

### Pattern 4: Controlled File Operations

**Use case**: Commands that should only read/edit specific types

```yaml
---
description: Update documentation
allowed-tools: [read_file, replace(*.md)]
---

Update documentation in @ $ARGUMENTS
```

**Note**: File pattern restrictions may not be supported in all versions.

## Real Examples from Official Docs

### Example 1: Git Commit Command

**Source**: Official Opencode documentation

```markdown
---
allowed-tools: run_shell_command(git add:*), run_shell_command(git status:*), run_shell_command(git commit:*)
description: Create a git commit
---

## Context

- Current git status: ! `git status`
- Current git diff (staged and unstaged changes): ! `git diff HEAD`
- Current branch: ! `git branch --show-current`
- Recent commits: ! `git log --oneline -10`

## Your task

Based on the above changes, create a single git commit.
```

**Allowed bash commands**:
- `git add .`
- `git add file.js`
- `git status`
- `git status --short`
- `git commit -m "message"`
- `git commit --amend`

**Blocked commands**:
- `rm file.js`
- `curl https://malicious.com`
- `npm install`
- Any non-git commands

### Example 2: Code Review (No Restrictions)

```markdown
---
description: Review this code for security vulnerabilities
---

Review this code for security vulnerabilities:
```

**No allowed-tools field** = All tools available

The agent can:
- Read files
- Write files
- Execute bash commands
- Use any tool

**Use when**: Command needs full flexibility

## When to Restrict Tools

### ✅ Restrict when:

1. **Security-sensitive operations**
   ```yaml
   # Git operations only
   allowed-tools: run_shell_command(git add:*), run_shell_command(git status:*)
   ```

2. **Focused tasks**
   ```yaml
   # Deep thinking only
   allowed-tools: think
   ```

3. **Read-only analysis**
   ```yaml
   # No modifications
   allowed-tools: [read_file, search_file_content, glob]
   ```

4. **Specific bash commands**
   ```yaml
   # Only npm scripts
   allowed-tools: run_shell_command(npm run test:*), run_shell_command(npm run build:*)
   ```

### ❌ Don't restrict when:

1. **Command needs flexibility**
   - Complex workflows
   - Exploratory tasks
   - Multi-step operations

2. **Tool needs are unpredictable**
   - General problem-solving
   - Debugging unknown issues

3. **Already in safe environment**
   - Sandboxed execution
   - Non-production systems

## Best Practices

### 1. Use Wildcards for Command Families

```yaml
# Good - allows all git commands
allowed-tools: run_shell_command(git *)

# Better - specific git operations
allowed-tools: run_shell_command(git add:*), run_shell_command(git status:*), run_shell_command(git commit:*)

# Best - minimal necessary permissions
allowed-tools: run_shell_command(git status:*), run_shell_command(git diff:*)
```

### 2. Combine Tool Types Appropriately

```yaml
# Analysis with optional git context
allowed-tools: [read_file, search_file_content, run_shell_command(git status:*)]
```

### 3. Test Restrictions

Create command and verify:
- Allowed operations work
- Blocked operations are prevented
- Error messages are clear

### 4. Document Why

```yaml
---
description: Create git commit (restricted to git commands only for security)
allowed-tools: run_shell_command(git add:*), run_shell_command(git status:*), run_shell_command(git commit:*)
---
```

## Tool Types

### File Operations
- `read_file` - Read files
- `write_file` - Write new files
- `replace` - Modify existing files
- `search_file_content` - Search file contents
- `glob` - Find files by pattern

### Execution
- `run_shell_command(pattern:*)` - Execute bash commands matching pattern
- `think` - Reasoning tool

### Other
- `delegate_to_agent` - Invoke sub-agents
- `google_web_search` - Search the web
- `web_fetch` - Fetch web pages

## Security Patterns

### Pattern: Prevent Data Exfiltration

```yaml
---
description: Analyze code locally
allowed-tools: [read_file, search_file_content, glob, think]
# No run_shell_command, web_fetch - cannot send data externally
---
```

### Pattern: Prevent Destructive Operations

```yaml
---
description: Review changes
allowed-tools: [read_file, run_shell_command(git diff:*), run_shell_command(git log:*)]
# No write_file, replace, git reset, git push --force
---
```

### Pattern: Controlled Deployment

```yaml
---
description: Deploy to staging
allowed-tools: run_shell_command(npm run deploy:staging), run_shell_command(git push origin:staging)
# Cannot deploy to production accidentally
---
```

## Limitations

1. **Wildcard patterns** may vary by version
2. **File-specific restrictions** (like `replace(*.md)`) may not be supported
3. **Cannot blacklist** - only whitelist
4. **All or nothing** for tool types - can't partially restrict

## Testing Tool Restrictions

### Verify Restrictions Work

1. Create command with restrictions
2. Try to use restricted tool
3. Confirm operation is blocked
4. Check error message

Example test:
```markdown
---
description: Test restrictions
allowed-tools: [read_file]
---

Try to write a file - this should fail.
```

Expected: Write operations blocked with error message.