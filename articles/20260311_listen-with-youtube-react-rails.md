---
title: "YouTube字幕同期アプリ「Listen With YouTube」をReact + Railsで作った"
emoji: "🎧"
type: "tech"
topics: ["react", "rails", "youtube", "yt-dlp"]
published: true
---

## はじめに

英語のYouTube動画を見ながらリスニング学習するとき、「字幕を見ながら、気になる単語の部分だけ聞き直したい」と思ったことはないだろうか。

既存のYouTube字幕機能では、単語単位でのシークやハイライトができない。そこで、URLを貼るだけで英語字幕を取得し、動画と同期表示する**Listen With YouTube**を作った。

この記事では、技術スタックの選定理由から実装のポイントまでを解説する。

## 完成したもの

- YouTube URLを貼り付けると、英語字幕を自動取得
- 動画再生に合わせて字幕をリアルタイムでハイライト
- **単語をクリックすると、その単語の再生位置にジャンプ**
- 再生中の単語がリアルタイムで強調表示される

## 技術スタック

| レイヤー | 技術 |
|---|---|
| フロントエンド | React 19 + Vite 7 |
| バックエンド | Ruby on Rails 8 (API-only) |
| 字幕取得 | yt-dlp |
| DB | SQLite3 |
| 動画プレイヤー | YouTube IFrame API |

## アーキテクチャ

```
[ブラウザ]
  ├── React App (Vite)
  │     ├── YouTube IFrame Player
  │     └── 字幕同期パネル
  │
  └── POST /api/transcript ──→ [Rails API]
                                   ├── TranscriptFetcher
                                   │     └── yt-dlp (字幕DL)
                                   ├── VttParser (VTT解析)
                                   └── Rails.cache (10分キャッシュ)
```

フロントエンドからは`POST /api/transcript`の1エンドポイントだけを叩く、シンプルな構成にした。

## バックエンド実装

### TranscriptFetcher: 字幕取得サービス

YouTube URLからvideo IDを抽出し、`yt-dlp`で英語字幕をダウンロードする。

```ruby
class TranscriptFetcher
  class InvalidUrl < StandardError; end
  class TranscriptNotFound < StandardError; end
  class FetchError < StandardError; end

  CACHE_TTL = 10.minutes

  def self.call(url)
    video_id = extract_video_id(url)
    raise InvalidUrl, "Invalid YouTube URL" unless video_id

    Rails.cache.fetch("transcript:#{video_id}", expires_in: CACHE_TTL) do
      captions = fetch_captions(url)
      { videoId: video_id, captions: captions }
    end
  end
end
```

**ポイント:**

- `youtube.com`、`youtu.be`、`/embed/`形式の3パターンに対応
- `Rails.cache.fetch`で同じ動画の再リクエストを10分間キャッシュ
- `yt-dlp`の実行には25秒のタイムアウトを設定

```ruby
def self.fetch_captions(url)
  Dir.mktmpdir("yt_subs") do |dir|
    cmd = [
      "yt-dlp",
      "--skip-download",        # 動画本体はDLしない
      "--write-sub",            # 手動字幕を取得
      "--write-auto-sub",       # 自動生成字幕も取得
      "--no-playlist",
      "--socket-timeout", "10",
      "--retries", "2",
      "--sub-lang", "en",       # 英語のみ
      "--sub-format", "vtt",    # VTT形式で出力
      "--output", File.join(dir, "%(id)s.%(ext)s"),
      url
    ]

    stdout, stderr, status = Timeout.timeout(25) { Open3.capture3(*cmd) }
    # ... エラーハンドリング
  end
end
```

`yt-dlp`は外部コマンドなので、`Open3.capture3`で実行し、タイムアウトとエラーハンドリングをきちんと行っている。

### VttParser: VTT字幕の解析

WebVTT形式の字幕ファイルをパースし、`[{ start: 秒数, end: 秒数, text: "..." }]`の配列に変換する。

```ruby
class VttParser
  CUE_TIME_REGEX = /(?<start>\d{2}:\d{2}:\d{2}\.\d{3})\s+-->\s+(?<end>\d{2}:\d{2}:\d{2}\.\d{3})/

  def self.parse(vtt_text)
    cues = []
    current = nil
    text_lines = []
    skip_section = false

    vtt_text.each_line do |line|
      stripped = line.strip
      # WEBVTT, NOTE, STYLE, REGION セクションをスキップ
      # タイムスタンプ行でキューを開始
      # テキスト行を収集
      # 空行でキューを確定
    end

    cues
  end

  def self.sanitize_text(text)
    cleaned = text.gsub(/<[^>]+>/, "")      # HTMLタグ除去
    cleaned = CGI.unescapeHTML(cleaned)       # HTMLエンティティデコード
    cleaned = cleaned.gsub("&nbsp;", "")     # &nbsp; 除去
    cleaned = cleaned.tr("\u00A0", " ")      # ノーブレークスペース変換
    cleaned.gsub(/\s+/, " ").strip           # 連続空白を正規化
  end
end
```

VTTファイルにはHTMLタグやエンティティが含まれることがあるので、`sanitize_text`メソッドで丁寧にクリーニングしている。

### API コントローラー

```ruby
# POST /api/transcript
# Request:  { "url": "https://www.youtube.com/watch?v=..." }
# Response: { "videoId": "abc123", "captions": [...] }
```

エラーハンドリングはHTTPステータスで適切に返す:
- **422**: 無効なURLまたはパラメータ不足
- **404**: 英語字幕が見つからない
- **502**: yt-dlpの実行失敗

## フロントエンド実装

### YouTube IFrame APIの初期化

```jsx
useEffect(() => {
  if (window.YT?.Player) {
    setYtReady(true)
    return
  }

  const script = document.createElement('script')
  script.src = 'https://www.youtube.com/iframe_api'
  window.onYouTubeIframeAPIReady = () => setYtReady(true)
  document.body.appendChild(script)
}, [])
```

YouTube IFrame APIは`<script>`タグを動的に追加し、`onYouTubeIframeAPIReady`コールバックで準備完了を検知する。

### リアルタイム字幕同期

250msごとにプレイヤーの再生位置を取得し、対応する字幕行を特定する。

```jsx
useEffect(() => {
  if (!playerRef.current || captions.length === 0) return

  const intervalId = window.setInterval(() => {
    if (!playerRef.current?.getCurrentTime) return
    const currentTime = playerRef.current.getCurrentTime()
    const index = captions.findIndex(
      (cue) => currentTime >= cue.start && currentTime < cue.end
    )
    setCurrentTime(currentTime)
    setActiveIndex(index)
  }, 250)

  return () => window.clearInterval(intervalId)
}, [captions, videoId])
```

250msのインターバルは、パフォーマンスと応答性のバランスが良い値だった。

### 単語レベルのシーク機能

最も工夫した部分がここ。字幕テキストを単語単位に分割し、各単語をクリック可能にしている。

```jsx
const handleSeekToWord = (cue, wordIndex, wordCount, event) => {
  event?.preventDefault()
  event?.stopPropagation()
  if (!playerRef.current?.seekTo || wordCount <= 0) return
  const duration = Math.max(0.01, cue.end - cue.start)
  const progress = Math.min(1, Math.max(0, wordIndex / wordCount))
  const backtrack = 0.1
  const seekTime = Math.max(0, cue.start + duration * progress - backtrack)
  playerRef.current.seekTo(seekTime, true)
}
```

**仕組み:**
1. 字幕テキストを空白で分割し、各単語に`wordIndex`を割り当て
2. クリックされた単語のインデックスから、字幕区間内での時間位置を計算
3. 0.1秒の「巻き戻しオフセット」を加えて、単語の出だしから聞けるように調整

### 単語レベルのハイライト

再生中のアクティブな字幕では、現在再生中の単語をリアルタイムで強調表示する。

```jsx
const duration = Math.max(0.01, cue.end - cue.start)
const progress = Math.min(1, Math.max(0, (currentTime - cue.start) / duration))
const currentWordIndex = Math.min(wordCount - 1, Math.floor(wordCount * progress))
```

再生位置の進捗率から現在の単語インデックスを計算し、`leading`（読み終わった単語）・`currentToken`（今の単語）・`trailing`（まだの単語）に分割してスタイリングしている。

### URLペースト時の自動読み込み

```jsx
const handlePaste = (event) => {
  const pasted = event.clipboardData?.getData('text')
  if (!pasted) return
  event.preventDefault()
  setUrl(pasted)
  fetchTranscript(pasted)
}
```

`onPaste`イベントでURLを検出し、ボタンを押さなくても自動的に字幕取得を開始する。ペーストするだけで即座に使えるUXにこだわった。

## 工夫した点・学び

### yt-dlpの扱い

`yt-dlp`は強力だが、外部コマンドなのでエラーパターンが多い。タイムアウト、リトライ制御、一時ディレクトリの利用など、堅牢性を意識して実装した。`Dir.mktmpdir`を使うことで、VTTファイルの後片付けも自動で行われる。

### VTTのサニタイズ

YouTubeの自動生成字幕には`<c>`タグや`&nbsp;`などが含まれることがある。表示崩れを防ぐために、HTMLタグ除去・エンティティデコード・空白正規化の3段階でクリーニングしている。

### 単語シークの精度

字幕の開始〜終了時間を単語数で等分するシンプルな方式だが、0.1秒のバックトラックを加えることで、体感的に「その単語から聞ける」UXを実現できた。

## まとめ

YouTube URLを貼るだけで英語字幕を同期表示し、単語クリックでシークできるアプリを、React + Rails + yt-dlpで構築した。

特に単語レベルのシーク＆ハイライト機能は、リスニング学習において「聞き逃した部分だけ聞き直す」体験を大幅に改善してくれる。

技術的には、外部コマンド（yt-dlp）の堅牢な呼び出し方や、VTT字幕のサニタイズ処理など、地味だが重要な実装パターンを多く含んでいる。
