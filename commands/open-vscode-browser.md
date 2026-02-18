---
description: VSCode内のBrowse Liteで指定URLを開く
disable-model-invocation: true
allowed-tools: Bash(python3:*), Bash(open:*), Bash(mkdir:*), Write, Read
argument-hint: [URL or 自然言語]
---

## Context

- Browse Lite拡張: !`ls -d ~/.vscode/extensions/antfu.browse-lite-* 2>/dev/null | head -1 || echo "未インストール"`
- URIハンドラ拡張: !`ls -d ~/.vscode/extensions/local.browse-lite-cli-* 2>/dev/null | head -1 || echo "未インストール"`

## Your task

引数 `$ARGUMENTS` を解析し、VSCode内のBrowse Lite（Chromiumベース埋め込みブラウザ）で指定URLを開く。
外部サイト（GitHub等）も表示可能。Cookieはデフォルトで永続化されるため、初回ログイン後はセッション維持される。

### 入力パターン

- `/sugano:open-vscode-browser https://github.com/org/repo/pull/99` → URL直接指定
- `/sugano:open-vscode-browser http://localhost:3000` → localhost
- `/sugano:open-vscode-browser` → 会話コンテキストからURLを推測して開く
- `/sugano:open-vscode-browser さっきのPR開いて` → コンテキストから該当URLを特定して開く

### Step 1: 前提チェック

Context セクションの結果から、以下の2つの拡張機能がインストール済みか確認する。

1. **Browse Lite** (`antfu.browse-lite`)
2. **カスタムURIハンドラ** (`local.browse-lite-cli`)

両方インストール済みなら Step 2 へ進む。
不足がある場合は、以下のセットアップ手順をユーザーに案内する。

#### Browse Lite が未インストールの場合

VSCode拡張機能マーケットプレイスから `antfu.browse-lite` をインストールするよう案内する。

#### カスタムURIハンドラが未インストールの場合

`~/.vscode/extensions/local.browse-lite-cli-1.0.0/` に以下の2ファイルを Write ツールで作成し、VSCodeのリロード（`Cmd+Shift+P` → `Developer: Reload Window`）を案内する。

`package.json`:
```json
{
  "name": "browse-lite-cli",
  "displayName": "Browse Lite CLI Opener",
  "description": "Open Browse Lite via URI handler from terminal",
  "version": "1.0.0",
  "publisher": "local",
  "engines": { "vscode": "^1.70.0" },
  "activationEvents": ["onUri"],
  "main": "./extension.js",
  "contributes": {}
}
```

`extension.js`:
```javascript
const vscode = require('vscode');
function activate(context) {
  context.subscriptions.push(
    vscode.window.registerUriHandler({
      handleUri(uri) {
        const params = new URLSearchParams(uri.query);
        const url = params.get('url');
        if (url) {
          vscode.commands.executeCommand('browse-lite.open', url);
        }
      }
    })
  );
}
function deactivate() {}
module.exports = { activate, deactivate };
```

### Step 2: URLの特定

`$ARGUMENTS` からURLを特定する:

- URLが直接指定されている場合はそのまま使用
- 自然言語や引数なしの場合は会話コンテキストからURLを推測

### Step 3: Browse Liteで開く

URLをパーセントエンコードし、カスタム拡張機能のURIハンドラ経由でBrowse Liteを開く:

```bash
ENCODED_URL=$(python3 -c "import urllib.parse; print(urllib.parse.quote('TARGET_URL', safe=''))")
open "vscode://local.browse-lite-cli/open?url=${ENCODED_URL}"
```

### Step 4: 結果の報告

開いたURLをユーザーに報告する。
