# CLAUDE.md

このファイルは、Claude Code がこのリポジトリで作業する際のガイダンスを提供します。

## プロジェクト概要

ccreader は GitHub Issues + Claude Code Action (Opus モデル) を使ったコンテンツ管理システムの実験的プロジェクトです。ユーザーが URL だけで Issue を作成すると、Claude が自動で処理します（コンテンツ取得、要約生成、ラベル付与、コメント投稿）。

## アーキテクチャ

```
Issue 作成 (URL のみ、`content` ラベル付き)
    ↓
GitHub Actions (claude-new-content.yml)
    ↓
Claude Code Action (model: opus)
    ↓
/ccreader:process-content コマンド (.claude/commands/ccreader/process-content.md)
    ↓
1. 「処理中」コメントを投稿
2. WebFetch でコンテンツ取得
3. 要約を生成
4. タイプ/トピックを判定（必要なら新規トピック作成）
5. 関連 Issue を検索（最大5件）
6. Issue を更新（タイトル、ラベル）
7. コメントを編集して要約を記載
```

## 主要ファイル

- `.claude/commands/ccreader/process-content.md` - メイン処理コマンド（`allowed-tools` 設定あり）
- `.github/workflows/claude-new-content.yml` - `content` ラベル付き Issue 作成時にトリガー
- `.github/labels.yml` - ラベル定義（type, topic）
- `.github/ISSUE_TEMPLATE/new-content.yml` - 新規コンテンツ用 Issue テンプレート

## ラベル

[.github/labels.yml](.github/labels.yml) を参照

- トピックはコンテンツの内容に応じて動的に作成される

## カスタムコマンド

- `/ccreader:process-content <repo> <issue_number>` - コンテンツ処理
- `/ccreader:sync-labels <repo>` - ラベル同期

## コミットメッセージ

[Conventional Commits](https://www.conventionalcommits.org/) に従い、スコープ付きで記述：

```
type(scope): Description

# 例:
feat(command): Add key-points extraction command
fix(workflow): Handle missing URL in issue body
docs(readme): Update setup instructions
refactor(command): Improve summary generation
```

- Types: `feat`, `fix`, `docs`, `refactor`, `chore`
- Scopes: `command`, `workflow`, `template`, `labels`
- Description: 現在形、大文字で開始
