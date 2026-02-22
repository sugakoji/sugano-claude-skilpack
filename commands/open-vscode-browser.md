---
description: VSCode内のBrowse Liteで指定URLを開く
disable-model-invocation: true
allowed-tools: Bash(echo:*), Read
argument-hint: [URL or 自然言語]
---

## Context

- Browse Lite拡張: !`ls -d ~/.vscode/extensions/sugakoji.browse-lite-* 2>/dev/null | head -1 || echo "未インストール"`
- BROWSE_LITE_IPC: !`echo "${BROWSE_LITE_IPC:-未設定}"`

## Your task

引数 `$ARGUMENTS` を解析し、VSCode内のBrowse Lite（Chromiumベース埋め込みブラウザ）で指定URLを開く。
外部サイト（GitHub等）も表示可能。Cookieはデフォルトで永続化されるため、初回ログイン後はセッション維持される。

### 入力パターン

- `/sugakoji:open-vscode-browser https://github.com/org/repo/pull/99` → URL直接指定
- `/sugakoji:open-vscode-browser http://localhost:3000` → localhost
- `/sugakoji:open-vscode-browser file:///Users/user/project/index.html` → ローカルファイル
- `/sugakoji:open-vscode-browser /Users/user/project/index.html` → ローカルファイルパス（`file://` を自動付与）
- `/sugakoji:open-vscode-browser` → 会話コンテキストからURLを推測して開く
- `/sugakoji:open-vscode-browser さっきのPR開いて` → コンテキストから該当URLを特定して開く

### Step 1: 前提チェック

Context セクションの結果から、以下を確認する。

1. **Browse Lite** (`sugakoji.browse-lite`) がインストール済みか
2. **`BROWSE_LITE_IPC`** 環境変数が設定されているか

両方OKなら Step 2 へ進む。不足がある場合は以下を案内する。

#### Browse Lite が未インストールの場合

VSCode拡張機能マーケットプレイスから `sugakoji.browse-lite` をインストールするよう案内する。

#### BROWSE_LITE_IPC が未設定の場合

以下を案内する:

1. VSCodeの統合ターミナルを開き直す（`BROWSE_LITE_IPC` はBrowse Lite拡張がターミナル起動時に自動設定する）
2. Claude Codeを再起動する（シェル環境変数を再読み込みするため）

### Step 2: URLの特定

`$ARGUMENTS` からURLを特定する:

- `http://` `https://` `file://` で始まるURLはそのまま使用
- `/` で始まる絶対パスの場合は `file://` を先頭に付与して `file:///path/to/file` 形式にする
- 自然言語や引数なしの場合は会話コンテキストからURLを推測

### Step 3: Browse Liteで開く

`BROWSE_LITE_IPC` が指すIPCファイルにURLを書き込む。Browse Lite拡張がファイル変更を検知してブラウザを開く。

```bash
echo "TARGET_URL" > "$BROWSE_LITE_IPC"
```

### Step 4: 結果の報告

開いたURLをユーザーに報告する。
