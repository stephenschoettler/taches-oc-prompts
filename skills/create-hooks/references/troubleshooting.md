# Troubleshooting

Common issues and solutions when working with hooks.

## Hook Not Triggering

### Symptom
Hook never executes, even when expected event occurs.

### Diagnostic steps

**1. Enable debug mode**
If available in your environment.

**2. Check hook file location**

Hooks must be in:
- Project: `.config/opencode/hooks.json`
- User: `~/.config/opencode/hooks.json`

Verify:
```bash
cat .config/opencode/hooks.json
# or
cat ~/.config/opencode/hooks.json
```

**3. Validate JSON syntax**

Invalid JSON is silently ignored:
```bash
jq . .config/opencode/hooks.json
```

If error: fix JSON syntax.

**4. Check matcher pattern**

Common mistakes:

❌ Case sensitivity
```json
{
  "matcher": "Bash"  // Won't match "run_shell_command"
}
```

✅ Fix
```json
{
  "matcher": "run_shell_command"
}
```

---

❌ Missing escape for regex
```json
{
  "matcher": "mcp__memory__*"  // Literal *, not wildcard
}
```

✅ Fix
```json
{
  "matcher": "mcp__memory__.*"  // Regex wildcard
}
```

**5. Test matcher in isolation**

```bash
node -e "console.log(/run_shell_command/.test('run_shell_command'))"  // true
```

### Solutions

**Missing hook file**: Create `.config/opencode/hooks.json` or `~/.config/opencode/hooks.json`

**Invalid JSON**: Use `jq` to validate and format:
```bash
jq . .config/opencode/hooks.json > temp.json && mv temp.json .config/opencode/hooks.json
```

**No matcher specified**: If you want to match all tools, omit the matcher field entirely:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "hooks": [...]  // No matcher = all tools
      }
    ]
  }
}
```

---

## Command Hook Failing

### Symptom
Hook executes but fails with error.

### Diagnostic steps

**1. Check debug output**

**2. Test command directly**

Copy the command and run in terminal:
```bash
echo '{"session_id":"test","tool_name":"run_shell_command"}' | /path/to/your/hook.sh
```

**3. Check permissions**
```bash
ls -l /path/to/hook.sh
chmod +x /path/to/hook.sh  // If not executable
```

**4. Verify dependencies**

Does the command require tools?
```bash
which jq  // Check if jq is installed
which osascript  // macOS only
```

### Common issues

**Missing executable permission**
```bash
chmod +x /path/to/hook.sh
```

**Missing dependencies**

Install required tools:
```bash
# macOS
brew install jq

# Linux
apt-get install jq
```

**Bad path**

Use absolute paths:
```json
{
  "command": "/Users/username/.config/opencode/hooks/script.sh"
}
```

Or use environment variables:
```json
{
  "command": "$OPENCODE_PROJECT_DIR/.config/opencode/hooks/script.sh"
}
```

**Timeout**

If command takes too long:
```json
{
  "command": "/path/to/slow-script.sh",
  "timeout": 120000  // 2 minutes
}
```

---

## Prompt Hook Not Working

### Symptom
Prompt hook blocks everything or doesn't block when expected.

### Diagnostic steps

**1. Check LLM response format**

Verify JSON is valid.

**2. Check prompt structure**

Ensure prompt is clear:
```json
{
  "prompt": "Evaluate: $ARGUMENTS\n\nReturn JSON: {\"decision\": \"approve\" or \"block\", \"reason\": \"why\"}"
}
```

**3. Test prompt manually**

Submit similar prompt to the agent directly to see response format.

### Common issues

**Ambiguous instructions**

❌ Vague
```json
{
  "prompt": "Is this ok? $ARGUMENTS"
}
```

✅ Clear
```json
{
  "prompt": "Check if this command is safe: $ARGUMENTS\n\nBlock if: contains 'rm -rf', 'mkfs', or force push to main\n\nReturn: {\"decision\": \"approve\" or \"block\", \"reason\": \"explanation\"}"
}
```

**Missing $ARGUMENTS**

❌ No placeholder
```json
{
  "prompt": "Validate this command"
}
```

✅ With placeholder
```json
{
  "prompt": "Validate this command: $ARGUMENTS"
}
```

**Invalid JSON response**

The LLM must return valid JSON. If it returns plain text, the hook fails.

Add explicit formatting instructions:
```
IMPORTANT: Return ONLY valid JSON, no other text:
{
  "decision": "approve" or "block",
  "reason": "your explanation"
}
```

---

## Hook Blocks Everything

### Symptom
Hook blocks all operations, even safe ones.

### Diagnostic steps

**1. Check hook logic**

Review the script/prompt logic. Is the condition too broad?

**2. Test with known-safe input**

```bash
echo '{"tool_name":"read_file","tool_input":{"file_path":"test.txt"}}' | /path/to/hook.sh
```

Expected: `{"decision": "approve"}`

**3. Check for errors in script**

Add error output:
```bash
#!/bin/bash
set -e  // Exit on error
input=$(cat)
# ... rest of script
```

### Solutions

**Logic error**

Review conditions:
```bash
# Before (blocks everything)
if [[ "$command" != "safe_command" ]]; then
  block
fi

# After (blocks dangerous commands)
if [[ "$command" == *"dangerous"* ]]; then
  block
fi
```

**Default to approve**

If logic is complex, default to approve on unclear cases:
```bash
# Default
decision="approve"
reason="ok"

# Only change if dangerous
if [[ "$command" == *"rm -rf"* ]]; then
  decision="block"
  reason="Dangerous command"
fi

echo "{\"decision\": \"$decision\", \"reason\": \"$reason\"}"
```

---

## Infinite Loop in Stop Hook

### Symptom
Stop hook runs repeatedly, the agent never stops.

### Cause
Hook blocks stop without checking `stop_hook_active` flag.

### Solution

**Always check the flag**:
```bash
#!/bin/bash
input=$(cat)
stop_hook_active=$(echo "$input" | jq -r '.stop_hook_active')

# If hook already active, don't block again
if [[ "$stop_hook_active" == "true" ]]; then
  echo '{"decision": undefined}'
  exit 0
fi

# Your logic here
if [ tests_passing ]; then
  echo '{"decision": "approve", "reason": "Tests pass"}'
else
  echo '{"decision": "block", "reason": "Tests failing"}'
fi
```

Or in prompt hooks:
```json
{
  "prompt": "Evaluate stopping: $ARGUMENTS\n\nIMPORTANT: If stop_hook_active is true, return {\"decision\": undefined}\n\nOtherwise check if tasks complete..."
}
```

---

## Hook Output Not Visible

### Symptom
Hook runs but output not shown to user.

### Cause
`suppressOutput: true` or output goes to stderr.

### Solutions

**Don't suppress output**:
```json
{
  "decision": "approve",
  "reason": "ok",
  "suppressOutput": false
}
```

**Use systemMessage**:
```json
{
  "decision": "approve",
  "reason": "ok",
  "systemMessage": "This message will be shown to user"
}
```

**Write to stdout, not stderr**:
```bash
echo "This is shown" >&1
echo "This is hidden" >&2
```

---

## Permission Errors

### Symptom
Hook script can't read files or execute commands.

### Solutions

**Make script executable**:
```bash
chmod +x /path/to/hook.sh
```

**Check file ownership**:
```bash
ls -l /path/to/hook.sh
chown $USER /path/to/hook.sh
```

**Use absolute paths**:
```bash
# Instead of
command="./script.sh"

# Use
command="$OPENCODE_PROJECT_DIR/hooks/script.sh"
```

---

## Hook Timeouts

### Symptom
Hook command timed out.

### Solutions

**Increase timeout**:
```json
{
  "type": "command",
  "command": "/path/to/slow-script.sh",
  "timeout": 300000  // 5 minutes
}
```

**Optimize script**:
- Reduce unnecessary operations
- Cache results when possible
- Run expensive operations in background

**Run in background**:
```bash
#!/bin/bash
# Start long operation in background
/path/to/long-operation.sh &

# Return immediately
echo '{"decision": "approve", "reason": "ok"}'
```

---

## Matcher Conflicts

### Symptom
Multiple hooks triggering when only one expected.

### Cause
Tool name matches multiple matchers.

### Diagnostic
Debug logs show multiple matches.

### Solutions

**Be more specific**:
```json
// Instead of
{"matcher": ".*"}  // Matches everything

// Use
{"matcher": "run_shell_command"}  // Exact match
```

**Check overlapping patterns**:
```json
{
  "hooks": {
    "PreToolUse": [
      {"matcher": "run_shell_command", ...},        // Matches run_shell_command
      {"matcher": "run_.*", ...},      // Also matches run_shell_command!
      {"matcher": ".*", ...}           // Also matches everything!
    ]
  }
}
```

Remove overlaps or make them mutually exclusive.

---

## Environment Variables Not Working

### Symptom
`$OPENCODE_PROJECT_DIR` or other variables are empty.

### Solutions

**Check variable spelling**:
- `$OPENCODE_PROJECT_DIR` (correct)
- `$CLAUDE_PROJECT_DIR` (wrong)

**Use double quotes**:
```json
{
  "command": "$OPENCODE_PROJECT_DIR/hooks/script.sh"
}
```

**In shell scripts, use from input**:
```bash
#!/bin/bash
input=$(cat)
cwd=$(echo "$input" | jq -r '.cwd')
cd "$cwd" || exit 1
```

---

## Debugging Workflow

**Step 1**: Enable debug mode (if available)

**Step 2**: Look for hook execution logs

**Step 3**: Test hook in isolation
```bash
echo '{"test":"data"}' | /path/to/hook.sh
```

**Step 4**: Check script with `set -x`
```bash
#!/bin/bash
set -x  // Print each command before executing
# ... your script
```

**Step 5**: Add logging
```bash
#!/bin/bash
echo "Hook started" >> /tmp/hook-debug.log
input=$(cat)
echo "Input: $input" >> /tmp/hook-debug.log
# ... your logic
echo "Decision: $decision" >> /tmp/hook-debug.log
```

**Step 6**: Verify JSON output
```bash
echo '{"decision":"approve","reason":"test"}' | jq .
```

If `jq` fails, JSON is invalid.