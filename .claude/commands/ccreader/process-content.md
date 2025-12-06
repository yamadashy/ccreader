---
allowed-tools: WebFetch, WebSearch, Bash(gh issue:*), Bash(gh label:*)
description: "新規コンテンツを処理（タイトル・要約・ラベル・コメントを自動生成）"
---

# 新規コンテンツ処理

## Context
- REPO: $1
- ISSUE_NUMBER: $2

## Task

URL だけが入力された Issue を処理し、コメントに要約を投稿してください。

**重要**: サマリー・タイトルなどの出力はすべて日本語で記述してください。

---

### 1. Issue 情報とラベル一覧の取得

```bash
gh issue view $2 --repo $1 --json title,body,labels
```
- Issue 本文から URL を抽出

```bash
gh label list --repo $1 --json name,description
```
- `type:*` と `topic:*` のラベルを確認し、後続の判定で使用

---

### 2. 進捗コメントを投稿

処理状況を示すコメントを投稿し、comment_id を取得します。

```bash
gh issue comment $2 --repo $1 --body "処理中...<img src=\"https://github.com/user-attachments/assets/5ac382c7-e004-429b-8e35-7feb3e8f9c6f\" width=\"14px\" height=\"14px\" style=\"vertical-align: middle; margin-left: 4px;\" />"
```

コメント投稿後、そのコメントのIDを取得：
```bash
gh api repos/$1/issues/$2/comments --jq '.[-1].id'
```

**以降、各ステップで進捗コメントを更新してください：**
```bash
gh api repos/$1/issues/comments/{comment_id} -X PATCH -f body="処理中... {現在の状況} <img src=\"https://github.com/user-attachments/assets/5ac382c7-e004-429b-8e35-7feb3e8f9c6f\" width=\"14px\" height=\"14px\" style=\"vertical-align: middle; margin-left: 4px;\" />"
```

例：
- `処理中... コンテンツを取得しています`
- `処理中... 要約を生成しています`
- `処理中... 関連 Issue を検索しています`

---

### 3. URL からコンテンツを取得
- WebFetch ツールでコンテンツを取得
- タイトル、本文、メタ情報を解析

---

### 4. 自動判定

すべて自動で判定してください：

#### タイトル
- 形式: `ページタイトル - サイト名`
- 例: `React 19の新機能まとめ - Zenn`
- サイト名が不明な場合はドメイン名を使用

#### タイプ（1つ選択）

ステップ1で取得した `type:*` ラベルから、コンテンツに最も適したものを1つ選択。
ラベルの description を参考に判断。

#### トピック（1-3個選択または新規作成）

ステップ1で取得した `topic:*` ラベルから、コンテンツに適したものを1-3個選択。
ラベルの description を参考に判断。

**適切なトピックがない場合**は、新しいトピックラベルを作成：
```bash
gh label create "topic:新しいトピック名" --repo $1 --color dddddd --description "トピックの説明（英語）"
```

#### 要約
- **要約だけで内容を理解できる詳細度**でまとめる
- コンテンツの要点を省略せずに記載
- 分かりやすく、読みたくなる構成で
- 箇条書きを活用

#### 読了時間
- 記事: 文字数から推定（日本語500字/分、英語200語/分）
- 動画: 再生時間
- 本: ページ数から推定

#### 公開日
- コンテンツから公開日・更新日を抽出
- 不明な場合は省略

#### 著者
- 著者名またはサイト名
- 不明な場合は省略

#### 対象読者・難易度
- 初心者向け / 中級者向け / 上級者向け
- 明確な場合のみ記載

#### 補足情報（任意）
コンテンツを読む上で役立つ補足情報があれば追加：
- 前提知識や関連する背景情報
- 専門用語の簡単な説明
- 必要に応じて **WebSearch** で調査して補足

---

### 5. 関連 Issue を検索

このコンテンツに関連する既存の Issue を最大5件検索します：

```bash
gh issue list --repo $1 --state all --search "関連キーワード" --json number,title --limit 10
```

検索のヒント：
- トピック、タイトル、要約から関連キーワードを抽出
- 複数の検索を試して関連度の高いものを選ぶ
- 自分自身（$2）は除外する

---

### 6. Issue タイトルを更新

```bash
gh issue edit $2 --repo $1 --title "コンテンツのタイトル"
```

---

### 7. ラベルを付与

```bash
gh issue edit $2 --repo $1 --add-label "type:xxx,topic:yyy,topic:zzz"
```

---

### 8. コメントを編集（要約と関連 Issue）

進捗コメントを最終的な内容に更新します：

```bash
gh api repos/$1/issues/comments/{comment_id} -X PATCH -f body="$(cat <<'EOF'
## 📖 要約

{要約}

- ポイント1
- ポイント2
- ポイント3

## 💡 補足

{補足情報があれば記載}

## ℹ️ 情報

- **公開日**: {YYYY年MM月}
- **著者**: {著者名}
- **タイプ**: {type}
- **トピック**: {topics}
- **読了時間**: 約 {N} 分
- **対象**: {初心者向け/中級者向け/上級者向け}

## 🔗 関連

- #12 関連Issueタイトル1
- #34 関連Issueタイトル2
EOF
)"
```

- 関連 Issue がない場合は Related セクションを省略
- 補足情報がない場合は Supplement セクションを省略
- 不明な Info 項目は省略

---

## Notes
- 元の URL は Issue 本文に残っているのでコメントには不要
- コンテンツ取得失敗時は URL とタイトルから推測
- 判定に迷ったらより一般的な選択肢を使用
- トピックは内容に最も適したものを選び、必要なら新規作成
