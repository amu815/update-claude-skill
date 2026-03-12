# update-claude-skill

A [Claude Code Skill](https://github.com/anthropics/claude-code) that installs a shell function to safely update Claude Code in nvm environments.

nvm環境でClaude Codeを安全にアップデートするシェル関数をインストールする [Claude Code スキル](https://github.com/anthropics/claude-code)です。

---

## Problem / 問題

When using [nvm](https://github.com/nvm-sh/nvm) to manage Node.js versions, running `npm install -g @anthropic-ai/claude-code` often fails with an `ENOTEMPTY` error:

[nvm](https://github.com/nvm-sh/nvm) でNode.jsのバージョンを管理している環境では、`npm install -g @anthropic-ai/claude-code` の実行時に `ENOTEMPTY` エラーが頻発します:

```
npm error code ENOTEMPTY
npm error syscall rename
npm error path /home/user/.nvm/versions/node/v22.x.x/lib/node_modules/@anthropic-ai/claude-code
```

This happens because npm tries to rename the existing package directory during an in-place upgrade, which can fail when files are still in use or due to filesystem race conditions.

これはnpmがインプレースアップグレード時に既存のパッケージディレクトリをリネームしようとし、ファイルが使用中だったりファイルシステムの競合状態が発生したりすると失敗するためです。

## Solution / 解決方法

The `update-claude` function works around this by:

`update-claude` 関数は以下の手順でこの問題を回避します:

1. Removing the old package directory completely (`rm -rf`) / 古いパッケージディレクトリを完全に削除（`rm -rf`）
2. Performing a clean install of the latest version / 最新バージョンをクリーンインストール

This is safe because `npm root -g` dynamically resolves the correct global package path for the current Node.js version.

`npm root -g` が現在のNode.jsバージョンに対応するグローバルパッケージパスを動的に解決するため、安全に動作します。

## Installation / インストール

### Via Claude Code Skills (recommended / 推奨)

```bash
npx skills add amu815/update-claude-skill -g -y
```

Then ask Claude Code: "install the update-claude skill" or invoke it with `/update-claude`.

インストール後、Claude Codeに「update-claudeスキルをインストールして」と伝えるか、`/update-claude` で呼び出してください。

### Manual installation (one-liner) / 手動インストール（ワンライナー）

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

## Usage / 使い方

After installation, simply run: / インストール後、以下を実行するだけです:

```bash
update-claude
```

This will: / 以下が実行されます:

1. Show the current Claude Code version / 現在のClaude Codeバージョンを表示
2. Remove the old package directory / 古いパッケージディレクトリを削除
3. Install the latest version / 最新バージョンをインストール
4. Confirm success with the new version number / 新しいバージョン番号で成功を確認

## License

MIT
