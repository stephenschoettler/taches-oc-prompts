# TÂCHES Prompts (OpenCode Port)

A port of the TÂCHES slash commands and prompt systems, adapted to work with [OpenCode](https://opencode.ai/) instead of Claude Code.

## Port Information

This is an unofficial port of the `glittercowboy/taches-cc-prompts` repository. All credit for the original prompts, architecture, and documentation belongs to the original author, TÂCHES.

This fork simply updates the `run-prompt.md` agent and `allowed-tools` to be compatible with OpenCode's sub-agent and tool system.

**Original Repository:** [https://github.com/glittercowboy/taches-cc-prompts](https://github.com/glittercowboy/taches-cc-prompts)

## Quick Start (for OpenCode)

```bash
# Clone this repo
git clone [https://github.com/stephenschoettler/taches-opencode-prompts.git](https://github.com/stephenschoettler/taches-opencode-prompts.git)
cd taches-opencode-prompts

# Create the OpenCode command directory (if it doesn't exist)
mkdir -p ~/.config/opencode/command/

# Install meta-prompting
cp meta-prompting/*.md ~/.config/opencode/command/

# Install todo management
cp todo-management/*.md ~/.config/opencode/command/

# Install context handoff
cp context-handoff/*.md ~/.config/opencode/command/
