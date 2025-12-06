# ccreader

GitHub Issue + Claude Code Action でコンテンツ（記事・動画・本など）を管理するリポジトリ

## 使い方

1. `New Issue` > `New Content` から記事などの URL を入力して Issue を作成
2. Claude Code Action で、タイトル・要約・ラベル付与・関連 Issue 検索
3. 読了したら Issue を Close

以下のようなURLをブックマークしておくと追加しやすい

https://github.com/yamadashy/ccreader/issues/new?template=new-content.yml

### セットアップ

1. リポジトリをフォーク
2. Claude Code で `/install-github-app` を実行し Secrets に登録
3. Claude Code で `/ccreader:sync-labels` を実行し、ラベルを同期

## 仕様

### 処理内容

Issue 作成時に Claude が自動で以下を実行：

1. URL からコンテンツを取得
2. タイトル・要約・補足情報を生成
3. タイプ・トピックを判定してラベル付与
4. 関連 Issue を検索（最大5件）
5. コメントに要約を投稿

### ラベル

[.github/labels.yml](.github/labels.yml) を参照

トピックはコンテンツの内容に応じて動的に追加される

## License

MIT
