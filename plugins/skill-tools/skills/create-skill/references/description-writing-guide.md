# Description 記述ガイド

## 公式

```
[何をするか] + [いつ使うか] + [トリガーフレーズ]
```

良い description には3つの要素がある。

1. **WHAT**: スキルの中核機能を説明する一文
2. **WHEN**: このスキルを起動すべき条件やシナリオ
3. **TRIGGERS**: ユーザーが呼び出す際に言いそうな具体的なフレーズ

## 制約

- 最大1024文字
- frontmatter に XML タグ（`<`, `>`）を含めない
- **常に三人称で書く**（例: "I can help you" や "You can use this to" ではなく "Processes files"）
- スキャンしやすさを保つ — Claude はこれを読んで、100以上あるスキルの中から起動すべきか判断する

## 良い例

```yaml
description: >-
  Generate a comprehensive code review for the current branch. Use when the
  user asks to "review my code", "check this PR", "audit changes", or
  "review for quality". Covers correctness, security, performance, and style.
```

```yaml
description: >-
  Create a new GitHub issue with structured labels and assignees. Use when
  user says "file an issue", "create a bug report", "open a feature request",
  or "track this problem".
```

## 悪い例

```yaml
# 曖昧すぎる — WHEN もトリガーもない
description: Helps with code

# 長すぎる — 重要な情報が埋もれる
description: >-
  This skill is designed to help users who want to perform code reviews.
  It can handle many different types of reviews including security reviews,
  performance reviews, and general code quality reviews. Users can invoke
  this skill whenever they need a review done on their code changes...

# XML を含む — frontmatter のパースが壊れる
description: Use <review> tags to trigger this skill
```

## コツ

- 最も重要な情報を先頭に置く
- 自然言語に合う具体的なトリガーフレーズを使う
- テスト: description だけを読んで、いつこのスキルを使うべきか正確にわかるか？
- フォーマルな言い方（"generate a code review"）とカジュアルな言い方（"check my code"）の両方を含める
