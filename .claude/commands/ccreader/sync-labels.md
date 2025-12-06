---
allowed-tools: Bash(gh label:*)
description: "labels.yml に定義されたラベルをリポジトリに同期"
---

# ラベル同期

## Context
- REPO: $1

## Task

`.github/labels.yml` に定義されたラベルをリポジトリに同期してください。

---

### 1. ラベル定義を確認

`.github/labels.yml` を読んで、定義されているラベルを確認。

---

### 2. ラベルを作成/更新

各ラベルに対して以下を実行：

```bash
gh label create "ラベル名" --repo $1 --color カラーコード --description "説明" --force
```

`--force` オプションで既存ラベルは上書き更新されます。

---

### 3. 完了報告

同期したラベルの一覧を報告してください。
