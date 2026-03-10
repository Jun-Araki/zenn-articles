---
title: "zshの歴史的背景から学ぶ：名前の由来とzshrcコマンド完全ガイド"
emoji: "🐚"
type: "tech"
topics: ["zsh", "shell", "terminal", "Linux"]
published: true
---

## はじめに

macOS のデフォルトシェルが Bash から **zsh** に切り替わったのは 2019 年の macOS Catalina からです。多くの開発者が毎日使っている zsh ですが、「そもそも zsh ってどういう意味？」「`.zshrc` の `rc` って何？」と聞かれると、答えに詰まる方も多いのではないでしょうか。

この記事では、zsh の歴史的背景から名称の由来を紐解き、`.zshrc` をはじめとする設定ファイルやよく使うコマンドまでを体系的に解説します。

---

## zsh の歴史

### 誕生：1990年、プリンストン大学にて

zsh は **1990 年**に、当時プリンストン大学の学生だった **Paul Falstad** によって開発されました。バージョン 1.0 は Usenet の `alt.sources` に投稿される形で公開されています。

当時のシェルの主流は以下のとおりです。

| シェル | 登場年 | 開発者 |
|--------|--------|--------|
| sh (Bourne Shell) | 1979 | Stephen Bourne |
| csh (C Shell) | 1979 | Bill Joy |
| tcsh | 1983 | Ken Greer |
| ksh (Korn Shell) | 1983 | David Korn |
| bash (Bourne Again Shell) | 1989 | Brian Fox |
| **zsh (Z Shell)** | **1990** | **Paul Falstad** |

zsh は bash・ksh・tcsh の優れた機能を取り込みつつ、独自の拡張を加えた「良いとこ取り」のシェルとして設計されました。

### 名前の由来：「Z」は何の略？

**zsh** の「Z」は、当時プリンストン大学の教授だった **Zhong Shao**（邵中）氏のログインID `zsh` に由来しています。Paul Falstad が Shao 氏の名前を「シェルの名前にちょうどいい」と考えて採用したもので、Shao 氏自身が開発に関わったわけではありません。

つまり、zsh = **Z** Shell であり、「Z」はアルファベットの最後の文字＝「究極のシェル」という意味を込めたという解釈もありますが、公式には人名由来です。

---

## 設定ファイルの名前と役割

### `.zshrc` の「rc」とは？

`rc` は **"Run Commands"** の略です。これは 1965 年の MIT の **CTSS**（Compatible Time-Sharing System）にまで遡ります。CTSS には `RUNCOM` という仕組みがあり、起動時に自動実行するコマンドリストを定義できました。この慣習が Unix に引き継がれ、`.bashrc`、`.vimrc`、`.zshrc` などの設定ファイル名に残っています。

### zsh の設定ファイル一覧

zsh はシェルの起動モードによって、読み込まれるファイルが異なります。

| ファイル | 読み込みタイミング | 主な用途 |
|----------|-------------------|----------|
| `.zshenv` | **常に**（すべてのzsh起動時） | 環境変数（`PATH` など） |
| `.zprofile` | ログインシェル起動時 | ログイン時の初期化（`.bash_profile` に相当） |
| `.zshrc` | **インタラクティブシェル**起動時 | エイリアス、プロンプト設定、プラグイン |
| `.zlogin` | ログインシェル起動時（`.zprofile` の後） | ログイン後に実行したい処理 |
| `.zlogout` | ログインシェル終了時 | クリーンアップ処理 |

#### 読み込み順序

```
ログインシェルの場合:
.zshenv → .zprofile → .zshrc → .zlogin

非ログインのインタラクティブシェルの場合:
.zshenv → .zshrc

スクリプト実行の場合:
.zshenv のみ
```

:::message
**ポイント**: `.zshenv` はすべてのケースで読まれるため、ここに重い処理を書くとスクリプトの実行速度に影響します。環境変数の設定のみに留めましょう。
:::

---

## .zshrc でよく使う設定

### 1. エイリアス（alias）

```bash
# 基本的なエイリアス
alias ll='ls -la'
alias la='ls -a'
alias ..='cd ..'
alias ...='cd ../..'

# Git 系
alias gs='git status'
alias ga='git add'
alias gc='git commit'
alias gp='git push'
alias gl='git log --oneline --graph'

# 安全装置
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
```

### 2. 環境変数とPATH

```bash
# PATH の追加（zsh 独自の配列記法）
typeset -U path  # 重複を自動除去
path=(
  $HOME/.local/bin
  /usr/local/bin
  $path
)

# エディタ設定
export EDITOR='vim'
export VISUAL='code'

# 言語設定
export LANG='ja_JP.UTF-8'
```

### 3. ヒストリ設定

```bash
HISTFILE=~/.zsh_history
HISTSIZE=10000
SAVEHIST=10000

# 便利なオプション
setopt HIST_IGNORE_DUPS      # 連続する重複コマンドを記録しない
setopt HIST_IGNORE_ALL_DUPS  # 過去の重複もすべて削除
setopt HIST_REDUCE_BLANKS    # 余分な空白を削除
setopt SHARE_HISTORY         # 複数ターミナル間でヒストリを共有
setopt HIST_IGNORE_SPACE     # スペースで始まるコマンドを記録しない
```

### 4. 補完（Completion）

zsh の補完システムは非常に強力で、bash との大きな差別化ポイントです。

```bash
# 補完システムの初期化
autoload -Uz compinit && compinit

# 補完オプション
setopt AUTO_MENU              # Tab連打でメニュー選択
setopt AUTO_COMPLETE          # 自動補完
zstyle ':completion:*' menu select  # 矢印キーで選択可能に

# 大文字小文字を区別しない補完
zstyle ':completion:*' matcher-list 'm:{a-z}={A-Z}'

# 補完候補にカラーを付ける
zstyle ':completion:*' list-colors ${(s.:.)LS_COLORS}
```

### 5. プロンプトのカスタマイズ

```bash
# シンプルなプロンプト
PROMPT='%F{green}%n@%m%f:%F{blue}%~%f$ '

# Git ブランチ表示付きプロンプト
autoload -Uz vcs_info
precmd() { vcs_info }
zstyle ':vcs_info:git:*' formats '%F{yellow}(%b)%f'
setopt PROMPT_SUBST
PROMPT='%F{green}%~%f ${vcs_info_msg_0_} $ '
```

プロンプトで使える主な変数：

| 変数 | 意味 |
|------|------|
| `%n` | ユーザー名 |
| `%m` | ホスト名 |
| `%~` | カレントディレクトリ（`~` 省略形） |
| `%d` | カレントディレクトリ（フルパス） |
| `%T` | 時刻（HH:MM） |
| `%F{色}...%f` | 文字色の指定 |

---

## zsh の便利な組み込み機能

### グロブ展開（Globbing）

zsh のグロブは bash より遥かに強力です。

```bash
# 再帰的グロブ（サブディレクトリも含む）
ls **/*.md

# 修飾子（Glob Qualifiers）
ls *(.)     # ファイルのみ
ls *(/)     # ディレクトリのみ
ls *(om)    # 更新日時順にソート
ls *(.mh-1) # 1時間以内に変更されたファイル
```

### ディレクトリスタック

```bash
setopt AUTO_PUSHD           # cd 時に自動で pushd
setopt PUSHD_IGNORE_DUPS    # 重複を無視

# 使い方
cd /usr/local    # スタックに追加
cd /etc          # スタックに追加
dirs -v          # スタックの一覧表示
cd ~2            # スタックの2番目に移動
```

### コマンドライン編集

```bash
# Emacs モード（デフォルト）
bindkey -e

# Vi モード
bindkey -v

# 便利なキーバインド追加
bindkey '^R' history-incremental-search-backward  # Ctrl+R で履歴検索
bindkey '^A' beginning-of-line                     # Ctrl+A で行頭
bindkey '^E' end-of-line                           # Ctrl+E で行末
```

---

## フレームワークとプラグインマネージャー

zsh のエコシステムには強力なフレームワークやプラグインマネージャーがあります。

### Oh My Zsh

最も有名な zsh フレームワーク。2009 年に Robby Russell が開発。テーマやプラグインが豊富です。

```bash
# インストール
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# .zshrc での設定例
plugins=(git docker node npm zsh-autosuggestions zsh-syntax-highlighting)
ZSH_THEME="robbyrussell"
```

### その他の選択肢

| ツール | 特徴 |
|--------|------|
| **Prezto** | Oh My Zsh より軽量 |
| **zinit** | 高速なプラグインマネージャー |
| **sheldon** | Rust製、高速 |
| **Starship** | クロスシェル対応のプロンプト |

---

## bash から zsh への移行ポイント

macOS ユーザーが bash から zsh に移行する際のポイントです。

```bash
# .bashrc / .bash_profile の内容を .zshrc に移行

# bash の設定
export PATH="$HOME/bin:$PATH"        # → そのまま使える
alias ll='ls -la'                     # → そのまま使える
source ~/.bash_completion             # → compinit に置き換え

# 注意点
# bash: 配列は 0 始まり
# zsh:  配列は 1 始まり（デフォルト）

# 互換性が必要な場合
setopt KSH_ARRAYS  # 0始まりに変更
```

---

## まとめ

| 項目 | 内容 |
|------|------|
| zsh の由来 | Zhong Shao 氏のログインIDに由来（1990年、プリンストン大学） |
| rc の意味 | "Run Commands"（1965年のCTSSに起源） |
| macOS デフォルト化 | 2019年（macOS Catalina）から |
| 最大の強み | 強力な補完、グロブ展開、カスタマイズ性 |

zsh は 30 年以上の歴史を持ちながら、今なお進化を続けるシェルです。名前の由来を知り、設定ファイルの意味を理解することで、より深く使いこなせるようになるでしょう。

ぜひ `.zshrc` を自分好みにカスタマイズして、日々のターミナル作業を効率化してみてください。
