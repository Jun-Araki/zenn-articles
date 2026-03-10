---
title: "Claude Codeの便利な機能・ショートカット完全ガイド"
emoji: "⌨️"
type: "tech"
topics: ["ClaudeCode", "AI", "CLI"]
published: true
---

## はじめに

**Claude Code** は Anthropic が提供する CLI ベースのAIコーディングアシスタントです。ターミナル上で動作し、ファイル読み書き・Git 操作・コード生成・デバッグなどをAIが自律的に行ってくれます。

しかし、機能が豊富すぎて「こんなことできたの？」と後から気づくことも多いのではないでしょうか。この記事では、Claude Code を使いこなすためのキーボードショートカット、スラッシュコマンド、CLI フラグ、そして設定・拡張機能までを網羅的に解説します。

---

## キーボードショートカット

### 基本操作

| ショートカット | 機能 |
|---------------|------|
| `Ctrl+C` | 現在の入力または生成をキャンセル |
| `Ctrl+D` | セッションを終了 |
| `Ctrl+L` | ターミナル画面をクリア（会話履歴は保持） |
| `Ctrl+O` | 詳細出力（verbose）の切り替え |
| `Ctrl+R` | コマンド履歴を逆順検索 |
| `Ctrl+G` | デフォルトのテキストエディタで入力を開く |
| `Ctrl+B` | 現在のタスクをバックグラウンドに送る |
| `Ctrl+F` | すべてのバックグラウンドエージェントを終了 |
| `Ctrl+T` | タスクリストの表示/非表示 |
| `Shift+Tab` | パーミッションモードの切り替え |

### モデル・思考モード

| ショートカット | 機能 |
|---------------|------|
| `Alt+P` / `Option+P` | 使用モデルの切り替え |
| `Alt+T` / `Option+T` | 拡張思考（Extended Thinking）の切り替え |

### 画像の貼り付け

| ショートカット | 機能 |
|---------------|------|
| `Ctrl+V` / `Cmd+V` / `Alt+V` | クリップボードから画像を貼り付け |

スクリーンショットを貼り付けて「このUIのバグを直して」と依頼する使い方が非常に便利です。

### 複数行入力

| ショートカット | 対応ターミナル |
|---------------|---------------|
| `\` + `Enter` | すべてのターミナル |
| `Option+Enter` | macOS デフォルト |
| `Shift+Enter` | iTerm2, WezTerm, Ghostty, Kitty |
| `Ctrl+J` | すべてのターミナル |

### テキスト編集（Emacs 系）

| ショートカット | 機能 |
|---------------|------|
| `Ctrl+A` | 行頭に移動 |
| `Ctrl+E` | 行末に移動 |
| `Ctrl+K` | 行末まで削除 |
| `Ctrl+U` | 行全体を削除 |
| `Ctrl+Y` | 削除したテキストを貼り付け |
| `Alt+B` | 1単語戻る |
| `Alt+F` | 1単語進む |

### 巻き戻し

| ショートカット | 機能 |
|---------------|------|
| `Esc` × 2回 | 直前の状態に巻き戻す、または会話を要約 |

Claude が意図しない変更をしたとき、`Esc` を2回押すとその操作を取り消せます。地味ですが最も重宝するショートカットの一つです。

---

## スラッシュコマンド

### セッション管理

| コマンド | 機能 |
|---------|------|
| `/clear` | 会話履歴をクリア（`/reset`、`/new` でも可） |
| `/resume [session]` | セッションを再開（`/continue` でも可） |
| `/fork [name]` | 現在の地点で会話をフォーク |
| `/rewind` | 前のチェックポイントに戻る |
| `/rename [name]` | セッション名を変更 |
| `/export [filename]` | 会話をテキストファイルにエクスポート |

### コンテキスト管理

| コマンド | 機能 |
|---------|------|
| `/context` | コンテキスト使用率をグリッド表示 |
| `/compact [指示]` | 会話を圧縮（オプションで焦点を指定可） |
| `/cost` | トークン使用統計を表示 |
| `/memory` | CLAUDE.md メモリファイルを編集 |

:::message
**コンテキスト管理のコツ**: 長時間のセッションでは `/compact` を定期的に使い、コンテキストウィンドウを節約しましょう。`/compact TypeScriptの型エラーに集中` のように焦点を指定すると、関連情報を優先的に残してくれます。
:::

### 設定・診断

| コマンド | 機能 |
|---------|------|
| `/config` | 設定画面を開く（`/settings` でも可） |
| `/doctor` | インストールと設定の診断 |
| `/help` | ヘルプを表示 |
| `/usage` | プランの使用制限を表示 |
| `/theme` | カラーテーマを変更 |

### Git・コードレビュー

| コマンド | 機能 |
|---------|------|
| `/diff` | インタラクティブな diff ビューア |
| `/pr-comments [PR]` | GitHub PR のコメントを取得 |
| `/security-review` | セキュリティ脆弱性のレビュー |

### 拡張機能

| コマンド | 機能 |
|---------|------|
| `/skills` | 利用可能なスキルを一覧表示 |
| `/mcp` | MCP サーバーを管理 |
| `/batch <指示>` | 大規模変更を並列実行 |
| `/hooks` | フック設定を管理 |

### その他

| コマンド | 機能 |
|---------|------|
| `/fast [on\|off]` | ファストモードの切り替え（同じモデルで高速出力） |
| `/vim` | Vim 編集モードの切り替え |
| `/copy` | 最後の応答をクリップボードにコピー |
| `/tasks` | バックグラウンドタスクの管理 |

---

## CLI フラグ（起動オプション）

### 基本的な起動方法

```bash
# インタラクティブモード
claude

# 初期プロンプト付きで起動
claude "このプロジェクトの構成を教えて"

# 最新の会話を継続
claude -c

# ワンショット実行（実行して終了）
claude -p "package.json の依存関係を一覧にして"
```

### モデル指定

```bash
claude --model claude-opus-4-6       # Opus 4.6 を使用
claude --model claude-sonnet-4-6     # Sonnet 4.6 を使用
claude --fast                        # ファストモードで起動
```

### パーミッション制御

```bash
# Plan モード（読み取り専用）で起動
claude --permission-mode plan

# ツールを制限して起動
claude --tools "Bash,Edit,Read"

# 特定コマンドのみ許可
claude --allowedTools "Bash(npm run *)" "Read"

# 特定コマンドを禁止
claude --disallowedTools "Bash(rm *)"
```

### システムプロンプト

```bash
# カスタムシステムプロンプト
claude --system-prompt "You are a Python expert"

# ファイルからシステムプロンプトを読み込み
claude --system-prompt-file ./custom-prompt.txt

# 追加のシステムプロンプト（既存に追記）
claude --append-system-prompt "常に TypeScript を使うこと"
```

### 出力形式（自動化用）

```bash
# JSON 出力
claude -p --output-format json "query"

# ストリーミング JSON
claude -p --output-format stream-json "query"

# 構造化出力（スキーマ指定）
claude -p --json-schema '{"type":"object"...}' "query"
```

### コスト管理

```bash
# 最大支出額を設定（USD）
claude -p --max-budget-usd 5.00 "大規模リファクタリング"

# ターン数を制限
claude -p --max-turns 3 "簡単な質問"
```

### Worktree（並列作業）

```bash
# 隔離された git worktree 内で実行
claude -w feature-auth

# 自動ブランチ名で worktree 作成
claude -w
```

:::message alert
**`--dangerously-skip-permissions`** というフラグもありますが、名前の通り危険です。CI/CD パイプラインなど、信頼された環境でのみ使用してください。
:::

---

## 設定ファイル

### 設定ファイルの優先順位

Claude Code の設定は3つのスコープで管理されます：

| ファイル | スコープ | 用途 |
|---------|---------|------|
| `~/.claude/settings.json` | ユーザー | 全プロジェクト共通の個人設定 |
| `.claude/settings.json` | プロジェクト | チーム共有の設定（Git管理） |
| `.claude/settings.local.json` | ローカル | 個人用のプロジェクト設定（`.gitignore` 推奨） |

### パーミッション設定

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run test *)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Bash(rm *)",
      "Edit(**/.env*)"
    ],
    "defaultMode": "acceptEdits"
  }
}
```

パーミッションモードは4種類あります：

| モード | 説明 |
|--------|------|
| `plan` | 読み取り専用。コード変更は行わない |
| `acceptEdits` | ファイル編集は自動許可、コマンド実行は確認 |
| `bypassPermissions` | すべて自動許可（危険） |
| `default` | 都度確認 |

### ステータスライン

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh"
  }
}
```

ステータスラインにコンテキスト使用率やGitブランチ情報を表示できます。

---

## CLAUDE.md ── プロジェクトメモリ

`CLAUDE.md` は Claude Code に永続的な指示を与えるためのファイルです。毎回のセッション開始時に自動的に読み込まれます。

### 配置場所

| 場所 | スコープ |
|------|---------|
| `CLAUDE.md`（プロジェクトルート） | プロジェクト全体 |
| `.claude/CLAUDE.md` | プロジェクト全体 |
| `~/.claude/CLAUDE.md` | ユーザー全体（全プロジェクト） |

### 書くべきこと

```markdown
# CLAUDE.md

## 基本ルール
- 応答は日本語で行う
- Git コミットメッセージは英語

## 環境
- Node.js: v24.13.0（Volta 管理）
- パッケージマネージャー: pnpm

## コーディング規約
- TypeScript strict モードを使用
- テストは Vitest で書く
```

### ルールの分割管理

大規模プロジェクトでは `.claude/rules/` ディレクトリで分割管理できます：

```
.claude/rules/
  ├── git.md           # Git ルール
  ├── typescript.md    # TypeScript 規約
  └── security.md      # セキュリティルール
```

---

## Hooks ── 自動化ワークフロー

Hooks は Claude Code のイベントに応じて自動実行される処理です。

### 主なイベント

| イベント | タイミング |
|---------|----------|
| `Stop` | Claude が応答完了時 |
| `PreToolUse` | ツール実行前（ブロック可能） |
| `PostToolUse` | ツール実行成功後 |
| `Notification` | 通知送信時 |
| `UserPromptSubmit` | ユーザーがプロンプト送信時 |
| `SessionStart` | セッション開始時 |

### 実装例：完了時にサウンドを鳴らす

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "afplay /System/Library/Sounds/Glass.aiff"
          }
        ]
      }
    ]
  }
}
```

### 実装例：ファイル保存時に自動フォーマット

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write $CLAUDE_FILE_PATH"
          }
        ]
      }
    ]
  }
}
```

---

## Skills ── 再利用可能な指示セット

Skills は特定のタスクに特化した指示をパッケージ化する仕組みです。

### スキルの作成

```bash
mkdir -p ~/.claude/skills/my-skill
```

`~/.claude/skills/my-skill/SKILL.md` を作成：

```yaml
---
name: my-skill
description: スキルの説明
---

ここにスキルの詳細な指示を書く...
```

### スキルの呼び出し

```
/my-skill 引数1 引数2
```

たとえば、記事作成スキルを用意しておけば `/zenn-article-writer 記事タイトル` のように一発で記事のテンプレート生成からGitプッシュまで自動化できます。

---

## MCP ── 外部ツール連携

**MCP（Model Context Protocol）** は、Claude Code を外部サービスやツールと接続するためのプロトコルです。

### サーバーの追加

```bash
# HTTP ベースのサーバー
claude mcp add --transport http notion https://mcp.notion.com/mcp

# ローカル stdio ベースのサーバー
claude mcp add --transport stdio airtable \
  --env AIRTABLE_API_KEY=YOUR_KEY \
  -- npx -y airtable-mcp-server
```

### 管理コマンド

```bash
claude mcp list       # 一覧表示
claude mcp get github # 詳細表示
claude mcp remove github  # 削除
```

### 人気の MCP サーバー

| サーバー | 用途 |
|---------|------|
| GitHub | PR・Issue 管理 |
| Sentry | エラー監視 |
| Notion | ドキュメント管理 |
| Slack | メッセージング |
| PostgreSQL | データベースクエリ |

---

## パワーユーザー向けテクニック

### 1. `!` プレフィックスで即座にコマンド実行

```
! npm test
! git status
```

プロンプト入力欄で `!` に続けてコマンドを書くと、Claude を介さず直接ターミナルコマンドを実行できます。

### 2. バックグラウンドタスクの活用

`Ctrl+B` でタスクをバックグラウンドに送れます。テスト実行中に別の質問をしたいときに便利です。

### 3. `/compact` の活用タイミング

コンテキストが 50% を超えたら `/compact` を検討しましょう。`/context` でいつでも使用率を確認できます。

### 4. Worktree で並列開発

```bash
claude -w feature-login   # ログイン機能を別ブランチで開発
claude -w fix-bug-123     # バグ修正を並行して作業
```

メインブランチを汚さずに、複数の機能を同時並行で開発できます。

### 5. Tab キーでサジェストを受け入れる

セッション開始時にグレーアウトされたコマンド候補が表示されることがあります。`Tab` キーでそのまま受け入れ可能です。

### 6. `/doctor` で問題診断

「なぜか動かない」と感じたら `/doctor` を実行。設定ミスや環境の問題を自動検出してくれます。

---

## トラブルシューティング

### よくある問題と解決法

| 問題 | 解決法 |
|------|--------|
| コンテキストが足りない | `/compact` で圧縮、または `/clear` で新規開始 |
| MCP サーバーに接続できない | `/mcp` で認証状態を確認、`/doctor` で診断 |
| パーミッションが毎回聞かれる | `settings.json` の `allow` に追加 |
| ファイル変更が反映されない | `Esc` × 2 で巻き戻してからやり直す |
| レスポンスが遅い | `/fast on` でファストモードに切り替え |

---

## まとめ

| カテゴリ | 覚えておきたいもの |
|---------|------------------|
| ショートカット | `Esc×2`（巻き戻し）、`Ctrl+B`（バックグラウンド）、`Shift+Tab`（パーミッション切替） |
| コマンド | `/compact`（圧縮）、`/context`（使用率）、`/doctor`（診断） |
| CLI フラグ | `-c`（継続）、`-p`（ワンショット）、`-w`（worktree） |
| 設定 | CLAUDE.md（永続指示）、Hooks（自動化）、Skills（再利用） |
| 拡張 | MCP でGitHub・Slack・DB等と連携 |

Claude Code は単なるAIチャットではなく、ターミナルに常駐する**開発パートナー**です。ショートカットや設定を使いこなすことで、開発効率は格段に上がります。

まずは `/context` でコンテキスト使用率を確認するところから始めてみてください。
