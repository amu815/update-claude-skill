# update-claude-skill

A [Claude Code Skill](https://github.com/anthropics/claude-code) that installs a shell function to safely update Claude Code in nvm environments.

## Problem

When using [nvm](https://github.com/nvm-sh/nvm) to manage Node.js versions, running `npm install -g @anthropic-ai/claude-code` often fails with an `ENOTEMPTY` error:

```
npm error code ENOTEMPTY
npm error syscall rename
npm error path /home/user/.nvm/versions/node/v22.x.x/lib/node_modules/@anthropic-ai/claude-code
```

This happens because npm tries to rename the existing package directory during an in-place upgrade, which can fail when files are still in use or due to filesystem race conditions.

## Solution

The `update-claude` function works around this by:

1. Removing the old package directory completely (`rm -rf`)
2. Performing a clean install of the latest version

This is safe because `npm root -g` dynamically resolves the correct global package path for the current Node.js version.

## Installation

### Via Claude Code Skills (recommended)

```bash
npx skills add amu815/update-claude-skill -g -y
```

Then ask Claude Code: "install the update-claude skill" or invoke it with `/update-claude`.

### Manual installation (one-liner)

```bash
RCFILE="${ZDOTDIR:-$HOME}/.$(basename "$SHELL")rc" && cat >> "$RCFILE" << 'FUNC'

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
FUNC
source "$RCFILE" && echo "update-claude function installed to $RCFILE"
```

## Usage

After installation, simply run:

```bash
update-claude
```

This will:
1. Show the current Claude Code version
2. Remove the old package directory
3. Install the latest version
4. Confirm success with the new version number

## License

MIT
