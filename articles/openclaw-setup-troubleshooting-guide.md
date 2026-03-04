---
title: "OpenClaw設定でつまずいたポイントまとめ"
emoji: "🦞"
type: "tech"
topics: ["OpenClaw", "ブラウザ自動化", "X投稿"]
published: false
---

# OpenClaw設定でつまずいたポイントまとめ

## このノートについて

この記事は、実際に Cursor 上で OpenClaw を触りながら、

- Zenn 記事リポジトリの構成
- OpenClaw Browser Relay（Chrome 拡張）
- Zenn 記事 → X 投稿までの自動化フロー

でつまずいたポイントを、あとから同じ沼にハマらないようにメモしておくための記録です。

## 1. Zenn の記事フォルダはどこに置くか問題

### 悩み

- `workspace` 直下にいろいろファイルがあり、
  - ルートに `articles/`
  - `zenn-content/` の中にも `articles/`
  - さらに途中で `zenn-articles/` も登場
- Zenn のデプロイ画面からは  
  「articles ディレクトリが見つかりません」  
  と怒られる。

### 解決

- **Zenn の正本リポジトリを 1 つに決める**。
  - 最終的に、`/Users/arakijun/.openclaw/workspace/zenn-articles` を正とした。
- その中をシンプルに:
  - `articles/` … 記事の Markdown
  - `books/` … 必要なら後で使う（空フォルダでも OK）
- それ以外の場所にある `articles/` は整理（削除・非推奨）し、  
  「**Zenn 関連は zenn-articles/ の中だけを見る**」と決めてしまうと頭がスッキリする。

## 2. OpenClaw Browser Relay のエラー

### 状況

- Chrome 右上のカニアイコンに赤い `!` が出ている。
- 拡張機能のオプションには:
  - Port: `18792`
  - 「Gateway token required」
- ドキュメントには「http://127.0.0.1:18792/ に接続」と書いてある。

### つまずきポイント

1. **Browser Relay サーバー本体が起動していないのに、拡張だけ先に触っていた。**
2. Gateway token をどこから取ってくるのか一瞬分かりにくい。

### Gateway token の取得

OpenClaw の設定ファイルから Gateway 用のトークンを確認できます。ターミナルで次を実行してください。

```bash
cat ~/.openclaw/openclaw.json
```

出力された JSON 内の `gateway.auth.token`（または環境変数 `OPENCLAW_GATEWAY_TOKEN` に相当する値）を、Chrome 拡張の「Gateway token」欄にそのまま貼り付けます。

### Mac / Windows の違い

Browser Relay には **Mac 版** と **Windows 版** があり、拡張のインストール方法やリレー起動の手順が環境によって異なります。公式ドキュメント（例: `docs.openclaw.ai/tools/chrome-extension`）で、自分の OS に合わせた手順を確認してください。

### 実際にやったこと

1. `openclaw tui` から Browser Relay / Gateway を起動する。
   - メニューから「browser extension」「gateway」系の項目を探して Start。
2. 上記のとおり `~/.openclaw/openclaw.json` から Gateway token を取り出す。
3. Chrome 拡張のオプション画面で:
   - Port はデフォルトの `18792` のまま。
   - Gateway token に上記トークンを貼り付けて Save。
4. そのタブでカニアイコンをクリックして、赤い `!` が消えれば接続完了。

ポイントは、

- **「拡張だけ」ではなく「OpenClaw 側のリレーサーバー」も起動が必要**
- ポートとトークンの 2 つが揃って初めて緑になる

と理解しておくこと。

## 3. Zenn 記事 → X 投稿までのフロー設計

### やりたいこと

- Zenn に記事を書く。
- その内容を元に X（Twitter）用の投稿文を生成する。
- できればそのままブラウザ自動操作で投稿まで持っていきたい。

### 用意したスキルたち

1. `zenn-article-writer`
   - `/zenn-articles/articles` に Zenn 用 Markdown を作るスキル。
   - FrontMatter や slug のルール（12〜50 文字、英数・ハイフン・アンダースコア）をここに明文化。
2. `zenn-to-x`
   - 記事ファイル（例: `changing_careers_to_engineer.md`）を読み、
   - 記事の要約＋読者メリットを意識した **日本語 280 文字以内の投稿案** を 1〜3 個生成する。
   - URL は `https://zenn.dev/jaraki/articles/<slug>` 形式で自動生成。
3. `x-post`
   - OpenClaw Browser Relay 経由で、実際に X の投稿画面を開き、
   - 生成したテキストを入力 → ユーザーに確認 → Post ボタンをクリックする。

### 実際につまずいたところ

- エージェントが「設定エラーでスキルが使えない」と言い張ることがあった。
  - 実際にはただのハルシネーションで、スキルファイルは単なるテキスト。
  - 「設定が足りない」と言われても、**出力された投稿案自体は十分使える**。
- 結局、最初は:
  1. `zenn-to-x` 相当のロジックで **文章だけ生成**。
  2. それをコピーして **手動で X に貼り付けて投稿**。
  という運用からスタートするのが現実的だと分かった。

## 4. これから同じことをやるなら

同じように OpenClaw で Zenn 記事と X 投稿を連携したいなら、次の順番がおすすめです。

1. **Zenn リポジトリを 1 箇所に決める**
   - 例: `~/.../zenn-articles`
   - その中に `articles/` と `books/` を作る。
2. **Browser Relay を動かせるようにする**
   - 自分の OS（Mac / Windows）に合った手順で拡張を導入する。
   - `openclaw tui` から Gateway / Relay を起動し、`cat ~/.openclaw/openclaw.json` で取得した token を拡張に設定する。
3. **スキルで役割分担する**
   - 記事を書くスキル（Zenn 用）
   - 記事から X 文面を作るスキル
   - ブラウザ経由で X に投稿するスキル
   に分け、最初は「生成まで／投稿は手動」にしておく。

## おわりに

OpenClaw と Zenn を組み合わせると、  
「記事執筆 → 要約 → X 投稿」までをかなりの部分まで自動化できますが、

- リポジトリ構成
- Browser Relay（ポートとトークン、Mac/Windows の違い）

の 2 点でつまずきやすいと感じました。

この記事が、未来の自分や同じ構成で悩んでいる誰かの助けになればうれしいです。

