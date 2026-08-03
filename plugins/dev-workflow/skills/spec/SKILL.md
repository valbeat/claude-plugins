---
name: spec
description: >-
  対話形式のプランニングセッションを通じて DESIGN.md と TODO.md を生成する。
  機能の計画やアーキテクチャ設計を行うとき、またはユーザーが
  "plan", "spec", "design", "create a spec" と発言したときに使う。
argument-hint: "[task description]"
---

# /spec - 対話的プランニングコマンド

このコマンドは、ユーザーのタスク説明をもとに、対話形式で DESIGN.md（設計書）と
TODO.md（タスクリスト）を生成する。

## 使い方

```bash
/spec Add OAuth2 to user authentication
/spec  # 対話形式で開始
```

---

## [1/5] タスク説明の準備

### タスク説明の取得

- `$ARGUMENTS` が存在する場合: そのまま使用
- 空の場合: AskUserQuestion で確認

### 既存ドキュメントの確認

Read ツールで `docs/DESIGN.md` と `docs/TODO.md` の存在を確認する。

既存ファイルが見つかった場合、AskUserQuestion で確認する:

```javascript
AskUserQuestion({
  questions: [
    {
      question: "既存のプランニングドキュメントが見つかりました。どう進めますか？",
      header: "Existing Docs",
      options: [
        { label: "新規作成", description: "既存ドキュメントを上書きする" },
        { label: "更新", description: "既存ドキュメントを読み込み、差分更新する" },
        { label: "キャンセル", description: "コマンドを終了する" }
      ],
      multiSelect: false
    }
  ]
})
```

---

## [2/5] DESIGN.md の生成

### コードベース分析

1. プロジェクト構造を把握する（Glob, Read）
2. 既存のアーキテクチャパターンを確認する
3. 関連する既存コードを読む

### DESIGN.md の作成

以下の構造で `docs/DESIGN.md` を作成する:

```markdown
# Design Document: [Task Name]

## Overview
[Purpose and background]

## Functional Requirements
- [ ] Requirement 1
- [ ] Requirement 2

## Non-Functional Requirements
- Performance
- Security
- Testability

## Architecture
[System structure, component diagram]

## Technology Choices
[Technologies, libraries to use]

## Impact Scope
[Files/modules affected by changes]
```

**品質要件 (MUST)**:
- Impact Scope: Glob/Read で存在を確認した実在のファイルパスのみを列挙する — パスを捏造しない
- Functional Requirements: 各項目は検証可能な記述にする
  （悪い例: "fast search" / 良い例: "search returns results for a 10k-record dataset"）
- Technology Choices: 新規追加が明示的にタスクの一部でない限り、プロジェクトの既存依存関係にあるライブラリのみを提案する
- 最終的なドキュメントに `[TBD]` のようなプレースホルダーを残さない — 未解決事項は interview フェーズに回す

### 設計レビュー (Fable エージェント)

DESIGN.md のドラフト作成後、ユーザー承認の**前に**、Fable エージェントで
設計レビューを行う。

起動する体数は budget tier で決める:

```bash
bash ~/.claude/skills/rate-pace/scripts/pace.sh tier   # -> L0 | L1 | L2
```

- `L2` … 2体を**並列**で起動する。1体目は下記のレビュー、2体目は
  「この設計を採用しない理由を探す」反証役（プロンプトは後述）
- それ以外（`L0` / `L1`、判定失敗・コマンド不在を含む）… 1体のみ

判定できない場合は必ず1体側に倒す。tier の詳細は `~/.claude/CLAUDE.md` の
**Budget Tier** を参照。

```
subagent_type: general-purpose
model: fable
prompt: |
  You are a skeptical software architect reviewing a design document before
  it reaches the user. Read docs/DESIGN.md and the files it references.

  Check:
  1. Are the trade-offs real? Flag any technology choice where a simpler
     alternative already in the project's dependencies would suffice.
  2. Is the Impact Scope complete? List files likely affected but missing
     (verify candidates exist with Glob before listing them).
  3. Is every functional requirement verifiable as written?
  4. What is the single riskiest assumption in this design?

  Return: a short list of concrete revisions with evidence,
  or exactly "Design is sound." if none.
```

`L2` のときは、上記と**並列**で反証役をもう1体起動する:

```
subagent_type: general-purpose
model: fable
prompt: |
  You are arguing against a design that is about to be approved. Read
  docs/DESIGN.md and the files it references.

  Your job is not to polish it but to find the strongest case for a
  DIFFERENT architecture. Specifically:
  1. Name one concrete alternative structure and say what it would make
     easier that this design makes hard.
  2. Identify the assumption that, if wrong, invalidates the most of this
     design. State how you would test it cheaply before committing.
  3. Point out anything this design makes irreversible. Reversible choices
     do not need this scrutiny; irreversible ones do.

  Ground every claim in the actual repository — cite files. Do not invent
  requirements the document does not state.

  Return: the alternative and its trade-off, or exactly
  "No stronger alternative found." if the design genuinely dominates.
```

指摘があれば DESIGN.md に反映してからユーザー承認に進む。
"Design is sound." の場合はそのまま進む。2体起動した場合、両者が食い違ったら
反証役の指摘を DESIGN.md の Trade-offs 節に明記した上で判断をユーザーに委ねる。

### ユーザー承認

AskUserQuestion で承認を得る:

```javascript
AskUserQuestion({
  questions: [
    {
      question: "DESIGN.md を生成しました。interview フェーズに進みますか？",
      header: "DESIGN.md Approval",
      options: [
        { label: "承認", description: "次のフェーズに進む" },
        { label: "却下", description: "コマンドを終了する" }
      ],
      multiSelect: false
    }
  ]
})
```

---

## [3/5] 深掘りインタビュー

### インタビューの実施

DESIGN.md の内容に基づき、AskUserQuestion で深掘りする:

**インタビューの観点**:
- 技術的な実装詳細
- UI/UX に関する考慮事項
- 懸念点とリスク
- トレードオフの判断

**重要なルール**:
- 自明な質問はしない
- 暗黙の前提や未決事項を掘り下げる
- ユーザーが「done」と言うまで続ける

### 仕様の更新

収集した情報を DESIGN.md に追記・更新する。

---

## [4/5] TODO.md の生成

### タスク分解

DESIGN.md に基づき、TDD サイクルに分解する:

```markdown
# Task List: [Task Name]

## Phase 1: [Feature Name]

- [ ] [RED] Write test: [Test content]
- [ ] [GREEN] Implement: [Implementation content]
- [ ] [REFACTOR] Refactor

## Phase 2: [Feature Name]

- [ ] [RED] Write test: [Test content]
- [ ] [GREEN] Implement: [Implementation content]
- [ ] [REFACTOR] Refactor
```

**品質要件 (MUST)**:
- 各 [RED] タスクには対象テストファイルのパスと検証内容を明記する
- 各 [GREEN] タスクには実装対象のファイル・関数を明記する
- フェーズは依存関係の順に並べる: 前のフェーズが後のフェーズに依存してはならない
- フェーズは小さく保つ: 1フェーズ = 独立してテスト可能な1つの振る舞い

### ユーザー承認

```javascript
AskUserQuestion({
  questions: [
    {
      question: "TODO.md を生成しました。このタスクリストで問題ありませんか？",
      header: "TODO.md Approval",
      options: [
        { label: "承認", description: "このタスクリストで完了とする" },
        { label: "却下", description: "コマンドを終了する" }
      ],
      multiSelect: false
    }
  ]
})
```

---

## [5/5] 完了と実装開始

### サマリー表示

```
✓ Planning completed

Generated files:
- docs/DESIGN.md  (Design document)
- docs/TODO.md    (Task list)
```

### 次のアクション

```javascript
AskUserQuestion({
  questions: [
    {
      question: "実装を開始しますか？",
      header: "Next Action",
      options: [
        { label: "実装を開始", description: "/impl を実行する" },
        { label: "終了", description: "計画のみで終了する" }
      ],
      multiSelect: false
    }
  ]
})
```

---

## 注意事項

### MUST ルール
- TDD 準拠: すべてのタスクをテストファーストで実装する
- Tidy First: 構造的変更と振る舞いの変更を分離する
- 不確実性への対処: 推測せず、質問する
