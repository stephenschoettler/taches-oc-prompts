# Hook Types and Events

Complete reference for all Opencode hook events.

## PreToolUse

**When it fires**: Before any tool is executed

**Can block**: Yes

**Input schema**:
```json
{
  "session_id": "abc123",
  "transcript_path": "~/.config/opencode/projects/.../session.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "PreToolUse",
  "tool_name": "run_shell_command",
  "tool_input": {
    "command": "npm install",
    "description": "Install dependencies"
  }
}
```

**Output schema** (to control execution):
```json
{
  "decision": "approve" | "block",
  "reason": "Explanation",
  "permissionDecision": "allow" | "deny" | "ask",
  "permissionDecisionReason": "Why",
  "updatedInput": {
    "command": "npm install --save-exact"
  }
}
```

**Use cases**:
- Validate commands before execution
- Block dangerous operations
- Modify tool inputs
- Log command attempts
- Ask user for confirmation

**Example**: Block force pushes to main
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "run_shell_command",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Check if this git command is safe: $ARGUMENTS\n\nBlock if: force push to main/master\n\nReturn: {\"decision\": \"approve\" or \"block\", \"reason\": \"explanation\"}"
          }
        ]
      }
    ]
  }
}
```

---

## PostToolUse

**When it fires**: After a tool completes execution

**Can block**: No (tool already executed)

**Input schema**:
```json
{
  "session_id": "abc123",
  "transcript_path": "~/.config/opencode/projects/.../session.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "PostToolUse",
  "tool_name": "write_file",
  "tool_input": {
    "file_path": "/path/to/file.js",
    "content": "..."
  },
  "tool_output": "File created successfully"
}
```

**Output schema**:
```json
{
  "systemMessage": "Optional message to display",
  "suppressOutput": false
}
```

**Use cases**:
- Auto-format code after write/edit
- Run tests after code changes
- Update documentation
- Trigger CI builds
- Send notifications

**Example**: Auto-format after edits
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "write_file|replace",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write $OPENCODE_PROJECT_DIR",
            "timeout": 10000
          }
        ]
      }
    ]
  }
}
```

---

## UserPromptSubmit

**When it fires**: User submits a prompt to the agent

**Can block**: Yes

**Input schema**:
```json
{
  "session_id": "abc123",
  "transcript_path": "~/.config/opencode/projects/.../session.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "UserPromptSubmit",
  "prompt": "Write a function to calculate factorial"
}
```

**Output schema**:
```json
{
  "decision": "approve" | "block",
  "reason": "Explanation",
  "systemMessage": "Message to user"
}
```

**Use cases**:
- Validate prompt format
- Block inappropriate requests
- Preprocess user input
- Add context to prompts
- Enforce prompt templates

**Example**: Require issue numbers in prompts
```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Check if prompt mentions an issue number (e.g., #123 or PROJ-456): $ARGUMENTS\n\nIf no issue number: {\"decision\": \"block\", \"reason\": \"Please include issue number\"}\nOtherwise: {\"decision\": \"approve\", \"reason\": \"ok\"}"
          }
        ]
      }
    ]
  }
}
```

---

## Stop

**When it fires**: Agent attempts to stop working

**Can block**: Yes

**Input schema**:
```json
{
  "session_id": "abc123",
  "transcript_path": "~/.config/opencode/projects/.../session.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "Stop",
  "stop_hook_active": false
}
```

**Output schema**:
```json
{
  "decision": "block" | undefined,
  "reason": "Why agent should continue",
  "continue": true,
  "systemMessage": "Additional instructions"
}
```

**Use cases**:
- Verify all tasks completed
- Check for errors that need fixing
- Ensure tests pass before stopping
- Validate deliverables
- Custom completion criteria

**Example**: Verify tests pass before stopping
```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "npm test && echo '{\"decision\": \"approve\"}' || echo '{\"decision\": \"block\", \"reason\": \"Tests failing\"}'"
          }
        ]
      }
    ]
  }
}
```

**Important**: Check `stop_hook_active` to avoid infinite loops. If true, don't block again.

---

## SubagentStop

**When it fires**: A sub-agent attempts to stop

**Can block**: Yes

**Input schema**:
```json
{
  "session_id": "abc123",
  "transcript_path": "~/.config/opencode/projects/.../session.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "SubagentStop",
  "stop_hook_active": false
}
```

**Output schema**: Same as Stop

**Use cases**:
- Verify sub-agent completed its task
- Check for errors in sub-agent output
- Validate sub-agent deliverables
- Ensure quality before accepting results

**Example**: Check if code-reviewer provided feedback
```json
{
  "hooks": {
    "SubagentStop": [
      {
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Review the subagent transcript: $ARGUMENTS\n\nDid the code-reviewer provide:\n1. Specific issues found\n2. Severity ratings\n3. Remediation steps\n\nIf missing: {\"decision\": \"block\", \"reason\": \"Incomplete review\"}\nOtherwise: {\"decision\": \"approve\", \"reason\": \"Complete\"}"
          }
        ]
      }
    ]
  }
}
```

---

## SessionStart

**When it fires**: At the beginning of a session

**Can block**: No

**Input schema**:
```json
{
  "session_id": "abc123",
  "transcript_path": "~/.config/opencode/projects/.../session.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "SessionStart",
  "source": "startup"
}
```

**Output schema**:
```json
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "Context to inject into session"
  }
}
```

**Use cases**:
- Load project context
- Inject sprint information
- Set environment variables
- Initialize state
- Display welcome messages

**Example**: Load current sprint context
```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "cat $OPENCODE_PROJECT_DIR/.sprint-context.txt | jq -Rs '{\"hookSpecificOutput\": {\"hookEventName\": \"SessionStart\", \"additionalContext\": .}}'"
          }
        ]
      }
    ]
  }
}
```

---

## SessionEnd

**When it fires**: When a session ends

**Can block**: No (cannot prevent session end)

**Input schema**:
```json
{
  "session_id": "abc123",
  "transcript_path": "~/.config/opencode/projects/.../session.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "SessionEnd",
  "reason": "exit" | "error" | "timeout" | "compact"
}
```

**Output schema**: None (hook output ignored)

**Use cases**:
- Save session state
- Cleanup temporary files
- Update logs
- Send analytics
- Archive transcripts

**Example**: Archive session transcript
```json
{
  "hooks": {
    "SessionEnd": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "cp $transcript_path $OPENCODE_PROJECT_DIR/.config/opencode/archives/$(date +%Y%m%d-%H%M%S).jsonl"
          }
        ]
      }
    ]
  }
}
```

---

## PreCompact

**When it fires**: Before context window compaction

**Can block**: Yes

**Input schema**:
```json
{
  "session_id": "abc123",
  "transcript_path": "~/.config/opencode/projects/.../session.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "PreCompact",
  "trigger": "manual" | "auto",
  "custom_instructions": "User's compaction instructions"
}
```

**Output schema**:
```json
{
  "decision": "approve" | "block",
  "reason": "Explanation"
}
```

**Use cases**:
- Validate state before compaction
- Save important context
- Custom compaction logic
- Prevent compaction at critical moments

---

## Notification

**When it fires**: Agent needs user input (awaiting response)

**Can block**: No

**Input schema**:
```json
{
  "session_id": "abc123",
  "transcript_path": "~/.config/opencode/projects/.../session.jsonl",
  "cwd": "/current/working/directory",
  "permission_mode": "default",
  "hook_event_name": "Notification"
}
```

**Output schema**: None

**Use cases**:
- Desktop notifications
- Sound alerts
- Status bar updates
- External notifications (Slack, etc.)

**Example**: macOS notification
```json
{
  "hooks": {
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Agent needs input\" with title \"Opencode\"'"
          }
        ]
      }
    ]
  }
}
```