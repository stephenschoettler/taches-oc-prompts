# Matchers and Pattern Matching

Complete guide to matching tools with hook matchers.

## What are matchers?

Matchers are regex patterns that filter which tools trigger a hook. They allow you to:
- Target specific tools (e.g., only `run_shell_command`)
- Match multiple tools (e.g., `write_file|replace`)
- Match tool categories (e.g., all MCP tools)
- Match everything (omit matcher)

---

## Syntax

Matchers use JavaScript regex syntax:

```json
{
  "matcher": "pattern"
}
```

The pattern is tested against the tool name using `new RegExp(pattern).test(toolName)`.

---

## Common Patterns

### Exact match
```json
{
  "matcher": "run_shell_command"
}
```
Matches: `run_shell_command`
Doesn't match: `Run_Shell_Command`

### Multiple tools (OR)
```json
{
  "matcher": "write_file|replace"
}
```
Matches: `write_file`, `replace`
Doesn't match: `read_file`, `run_shell_command`

### Starts with
```json
{
  "matcher": "^run_"
}
```
Matches: `run_shell_command`
Doesn't match: `read_file`

### Ends with
```json
{
  "matcher": "_file$"
}
```
Matches: `read_file`, `write_file`
Doesn't match: `run_shell_command`, `replace`

### Contains
```json
{
  "matcher": ".*write.*"
}
```
Matches: `write_file`
Doesn't match: `read_file`, `replace`

Case-sensitive! `write` won't match `Write`.

### Any tool (no matcher)
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "hooks": [...]  // No matcher = matches all tools
      }
    ]
  }
}
```

---

## Tool Categories

### All file operations
```json
{
  "matcher": "read_file|write_file|replace|glob|search_file_content"
}
```

### All bash tools
```json
{
  "matcher": "run_shell_command"
}
```

### All MCP tools
```json
{
  "matcher": "mcp__.*"
}
```
Matches: `mcp__memory__store`, `mcp__filesystem__read`, etc.

### Specific MCP server
```json
{
  "matcher": "mcp__memory__.*"
}
```
Matches: `mcp__memory__store`, `mcp__memory__retrieve`
Doesn't match: `mcp__filesystem__read`

### Specific MCP tool
```json
{
  "matcher": "mcp__.*__write.*"
}
```
Matches: `mcp__filesystem__write`, `mcp__memory__write`
Doesn't match: `mcp__filesystem__read`

---

## MCP Tool Naming

MCP tools follow the pattern: `mcp__{server}__{tool}`

Examples:
- `mcp__memory__store`
- `mcp__filesystem__read`
- `mcp__github__create_issue`

**Match all tools from a server**:
```json
{
  "matcher": "mcp__github__.*"
}
```

**Match specific tool across all servers**:
```json
{
  "matcher": "mcp__.*__read.*"
}
```

---

## Real-World Examples

### Log all bash commands
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "run_shell_command",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.command' >> ~/bash-log.txt"
          }
        ]
      }
    ]
  }
}
```

### Format code after any file write
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "write_file|replace",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write $OPENCODE_PROJECT_DIR"
          }
        ]
      }
    ]
  }
}
```

### Validate all MCP memory writes
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__memory__.*",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Validate this memory operation: $ARGUMENTS\n\nCheck if data is appropriate to store.\n\nReturn: {\"decision\": \"approve\" or \"block\", \"reason\": \"why\"}"
          }
        ]
      }
    ]
  }
}
```

### Block destructive git commands
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "run_shell_command",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/check-git-safety.sh"
          }
        ]
      }
    ]
  }
}
```

`check-git-safety.sh`:
```bash
#!/bin/bash
input=$(cat)
command=$(echo "$input" | jq -r '.tool_input.command')

if [[ "$command" == *"git push --force"* ]] || \
   [[ "$command" == *"rm -rf /"* ]] || \
   [[ "$command" == *"git reset --hard"* ]]; then
  echo '{"decision": "block", "reason": "Destructive command detected"}'
else
  echo '{"decision": "approve", "reason": "Safe"}'
fi
```

---

## Multiple Matchers

You can have multiple matcher blocks for the same event:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "run_shell_command",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/bash-validator.sh"
          }
        ]
      },
      {
        "matcher": "write_file|replace",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/file-validator.sh"
          }
        ]
      },
      {
        "matcher": "mcp__.*",
        "hooks": [
          {
            "type": "command",
            "command": "/path/to/mcp-logger.sh"
          }
        ]
      }
    ]
  }
}
```

Each matcher is evaluated independently. A tool can match multiple matchers.

---

## Debugging Matchers

### Enable debug mode (if supported)
```bash
# Debug mode flag if applicable
```

### Test your matcher

Use JavaScript regex to test patterns:

```javascript
const toolName = "mcp__memory__store";
const pattern = "mcp__memory__.*";
const regex = new RegExp(pattern);
console.log(regex.test(toolName)); // true
```

Or in Node.js:
```bash
node -e "console.log(/mcp__memory__.*/.test('mcp__memory__store'))"
```

### Common mistakes

❌ **Case sensitivity**
```json
{
  "matcher": "Bash"  // Won't match "run_shell_command"
}
```

✅ **Correct**
```json
{
  "matcher": "run_shell_command"
}
```

---

❌ **Missing escape**
```json
{
  "matcher": "mcp__memory__*"  // * is literal, not wildcard
}
```

✅ **Correct**
```json
{
  "matcher": "mcp__memory__.*"  // .* is regex for "any characters"
}
```

---

## Advanced Patterns

### Negative lookahead (exclude tools)
```json
{
  "matcher": "^(?!read_file).*"
}
```
Matches: Everything except `read_file`

### Match any file operation except search
```json
{
  "matcher": "^(read_file|write_file|replace|glob)$"
}
```

---

## Performance Considerations

**Broad matchers** (e.g., `.*`) run on every tool use:
- Simple command hooks: negligible impact
- Prompt hooks: can slow down significantly

**Recommendation**: Be as specific as possible with matchers to minimize unnecessary hook executions.

**Example**: Instead of matching all tools and checking inside the hook:
```json
{
  "matcher": ".*",  // Runs on EVERY tool
  "hooks": [
    {
      "type": "command",
      "command": "if [[ $(jq -r '.tool_name') == 'run_shell_command' ]]; then ...; fi"
    }
  ]
}
```

Do this:
```json
{
  "matcher": "run_shell_command",  // Only runs on run_shell_command
  "hooks": [
    {
      "type": "command",
      "command": "..."
    }
  ]
}
```

---

## Tool Name Reference

Common Opencode tool names:
- `run_shell_command`
- `read_file`
- `write_file`
- `replace`
- `glob`
- `search_file_content`
- `delegate_to_agent`
- `google_web_search`
- `web_fetch`

MCP tools: `mcp__{server}__{tool}` (varies by installed servers)