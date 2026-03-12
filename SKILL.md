---
name: update-claude
description: Install a shell function that safely updates Claude Code in nvm environments by avoiding ENOTEMPTY errors
metadata:
  author: amu815
  version: 1.0.0
---

# update-claude Skill

## What this skill does

When invoked, install the `update-claude` shell function into the user's shell configuration file. This function safely updates Claude Code (`@anthropic-ai/claude-code`) in nvm environments by removing the old package directory before reinstalling, which avoids the common `ENOTEMPTY` error.

## Instructions

1. Detect the user's shell by checking `$SHELL` (bash or zsh).
2. Determine the rc file path:
   - For zsh: `${ZDOTDIR:-$HOME}/.zshrc`
   - For bash: `$HOME/.bashrc`
3. Check if the `update-claude` function already exists in the rc file. If it does, inform the user and skip.
4. Append the following function to the rc file:

```bash

# Claude Code updater - avoid ENOTEMPTY errors in nvm environments
update-claude() {
  echo "Current version: $(claude --version 2>/dev/null || echo 'unknown')"
  local pkg_dir="$(npm root -g)/@anthropic-ai/claude-code"
  echo "Package path: $pkg_dir"
  echo "Removing old version..."
  rm -rf "$pkg_dir"
  echo "Installing latest version..."
  npm install -g @anthropic-ai/claude-code@latest
  if [ $? -eq 0 ]; then
    echo "Update complete! Version: $(claude --version)"
  else
    echo "Installation failed. Check error logs."
    return 1
  fi
}
```

5. Source the rc file to make the function immediately available.
6. Confirm success to the user by showing which file was modified.

## Usage after installation

The user can run `update-claude` in their terminal at any time to safely update Claude Code.
