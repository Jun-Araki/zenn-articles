---
title: "初めてのClaude Codeセットアップ"
emoji: "🤖"
type: "tech"
topics: ["claude", "ClaudeCode", "AI", "開発環境"]
published: true
---

# 初めてのClaude Codeセットアップ

Anthropic が提供する CLI ツール **Claude Code** を使うと、ターミナルから直接 Claude と対話しながらコードを書いたり、ファイルを編集したりできます。この記事では、macOS 環境での初回セットアップ手順を解説します。

## Claude Code とは？

Claude Code は Anthropic 公式の CLI（コマンドラインインターフェース）で、以下のことができます。

- ファイルの読み書き・編集
- シェルコマンドの実行
- Git 操作
- コードのレビュー・デバッグ
- ドキュメント作成

ブラウザを開かずに、ターミナル上で AI とペアプログラミングができるのが最大の特徴です。

## 必要なもの

- macOS（または Linux / WSL）
- Node.js v18 以上
- Anthropic のアカウント（[claude.ai](https://claude.ai) でサインアップ）

## インストール手順

### 1. Node.js のインストール確認

```bash
node -v
# v18.0.0 以上であれば OK
```

Node.js が入っていない場合は [Volta](https://volta.sh/) や [nvm](https://github.com/nvm-sh/nvm) でインストールするのがおすすめです。

```bash
# Volta を使う場合
curl https://get.volta.sh | bash
volta install node
```

### 2. Claude Code のインストール

```bash
npm install -g @anthropic-ai/claude-code
```

インストール後、バージョンを確認します。

```bash
claude --version
```

### 3. 認証

```bash
claude
```

初回起動時にブラウザが開き、Anthropic アカウントでの OAuth 認証が求められます。ログイン後、ターミナルに戻ると認証完了です。

## 基本的な使い方

プロジェクトディレクトリに移動してから `claude` を起動します。

```bash
cd ~/Project/my-app
claude
```

あとは日本語でそのまま話しかけるだけです。

```
> このファイルのバグを直して
> テストを書いて
> README を更新して
```

### 便利なオプション

| コマンド | 説明 |
|---------|------|
| `claude` | 対話モードで起動 |
| `claude "質問内容"` | 1 回だけ質問して終了 |
| `claude --print "質問"` | 結果をそのままターミナルに出力 |

## CLAUDE.md でカスタマイズ

プロジェクトルートに `CLAUDE.md` を置くと、Claude Code がプロジェクト固有のルールを読み込んでくれます。

```markdown
# CLAUDE.md

## 開発環境
- Ruby 3.2 / Rails 8
- テストは RSpec を使うこと

## コーディング規約
- インデントはスペース 2 つ
- コメントは日本語で書くこと
```

これにより、毎回同じ説明をしなくて済むようになります。

## ~/.claude/commands/ でカスタムコマンド

`~/.claude/commands/` ディレクトリに `.md` ファイルを置くと、`/ファイル名` でいつでも呼び出せるカスタムコマンドが作れます。

```
~/.claude/commands/
└── zenn-article-writer.md  # /zenn-article-writer で呼び出せる
```

よく使うワークフローをコマンドとして登録しておくと非常に便利です。

## まとめ

Claude Code のセットアップはとてもシンプルで、`npm install -g` の 1 コマンドで完了します。`CLAUDE.md` や `~/.claude/commands/` を活用することで、自分の開発スタイルに合った AI アシスタント環境を作れます。

ぜひ日常の開発ワークフローに取り入れてみてください。
