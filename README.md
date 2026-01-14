# TÂCHES Prompts (Opencode Port)

A port of the TÂCHES slash commands and prompt systems, adapted to work with [Opencode](https://opencode.ai/) instead of Claude Code.

## Port Information

This is an unofficial port of the `glittercowboy/taches-cc-prompts` repository. All credit for the original prompts, architecture, and documentation belongs to the original author, TÂCHES.

This fork updates the agents, tools, and paths to be compatible with Opencode's sub-agent and tool system.

**Original Repository:** [https://github.com/glittercowboy/taches-cc-prompts](https://github.com/glittercowboy/taches-cc-prompts)

## Quick Start (for Opencode)

```bash
# Clone this repo
git clone https://github.com/stephenschoettler/taches-opencode-prompts.git
cd taches-opencode-prompts

# Create the Opencode command directory (if it doesn't exist)
mkdir -p ~/.config/opencode/commands/

# Install meta-prompting commands
cp commands/create-meta-prompt.md ~/.config/opencode/commands/
cp commands/create-prompt.md ~/.config/opencode/commands/
cp commands/run-prompt.md ~/.config/opencode/commands/

# Install todo management commands
cp commands/add-to-todos.md ~/.config/opencode/commands/
cp commands/check-todos.md ~/.config/opencode/commands/

# Install context handoff command
cp commands/whats-next.md ~/.config/opencode/commands/
```