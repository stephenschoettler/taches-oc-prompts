# Context Handoff for OpenCode

Create structured handoff documents to continue work in a fresh context without losing progress.

## The Problem

You're deep in a conversation, made significant progress, but need to start a fresh chat (context getting full, switching tasks, or just want a clean slate). You lose all the context about what was done, what remains, and why decisions were made.

## The Solution

`/whats-next` analyzes the current conversation and creates a structured handoff document (`whats-next.md`) in your working directory. In your next chat, reference it with `@whats-next.md` to continue seamlessly.

## Command

### `/whats-next`

Analyzes the conversation and creates a handoff document with:

1. **Original task** - What was initially requested
2. **Work completed** - What's been done toward that task
3. **Work remaining** - What still needs finishing (no scope creep)
4. **Context** - Decisions, approaches, blockers, gotchas

**Output format:**
```xml
<original_task>
Implement user authentication with JWT tokens
</original_task>

<work_completed>
- Created auth middleware in middleware/auth.ts:1-45
- Added JWT signing/verification in utils/jwt.ts:1-67
- Updated User model with password hashing in models/User.ts:23-34
- Created login endpoint in routes/auth.ts:12-56
</work_completed>

<work_remaining>
- Add token refresh endpoint
- Implement logout (token blacklist)
- Add password reset flow
- Write tests for auth middleware
</work_remaining>

<context>
- Using jsonwebtoken library (already in dependencies)
- Tokens expire in 24h (configurable via JWT_EXPIRY env var)
- Password hashing uses bcrypt with 10 rounds
- Discovered: User.findByEmail returns null for missing users (handle this in routes)
</context>
```

## Why This Works

**Preserves progress:**
- Exact file paths and line numbers
- Specific work completed, not vague summaries
- Clear remaining tasks tied to original goal

**Prevents scope creep:**
- Focuses only on completing the original request
- Doesn't add new features or "nice to haves"
- Maintains task boundaries across context switches

**Transfers critical decisions:**
- Why certain approaches were chosen
- Technical constraints discovered
- Gotchas to avoid

## Installation

**Install globally** - works in any directory:

```bash
cp whats-next.md ~/.config/opencode/command/
```

The command works everywhere. Each project gets its own `whats-next.md` in the working directory.

## Usage

**End of current conversation:**
```
You: /whats-next
OpenCode: [Analyzes conversation, writes whats-next.md]
✓ Created whats-next.md - reference with @whats-next.md in a new chat to continue
```

**Start of new conversation:**
```
You: @whats-next.md continue this work
OpenCode: [Reads handoff document, understands context, resumes work]
```

## Example Workflow

**Conversation 1 (getting full):**
```
You: "Build a user dashboard with real-time analytics"
OpenCode: [Works on task, creates components, sets up data fetching...]
You: /whats-next

✓ Created whats-next.md
```

**Conversation 2 (fresh context):**
```
You: @whats-next.md finish this
OpenCode: [Reads whats-next.md]
"I see we've completed the dashboard layout and real-time data
connection. Still need to add the chart components and error handling.
Continuing from src/components/Dashboard.tsx:67..."
```

## File Structure

**Global (install once):**
```
~/.config/opencode/command/
  whats-next.md         # Command
```

**Per-project (created when needed):**
```
/your/project/
  whats-next.md         # Handoff document for this project

/another/project/
  whats-next.md         # Different project's handoff
```

## Tips

- Use `/whats-next` when context feels full or cluttered
- Reference with `@whats-next.md` in new chats for seamless continuation
- The file is overwritten each time - it's a snapshot, not a log
- Delete `whats-next.md` when work is complete
- Works great with `/add-to-todos` for long-term task tracking

## Integration

**Combine with todo management:**
- `/whats-next` for immediate continuation (same session/day)
- `/add-to-todos` for longer-term backlog (weeks/months)

Use whats-next.md for "I'll finish this after lunch" and TO-DOS.md for "I'll get to this eventually."

---

**Questions or improvements?** Open an issue or submit a PR.

—TÂCHES
