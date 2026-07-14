---
name: interview
description: >-
  DESIGN.md を読み込み、技術的な実装・懸念点・トレードオフについて深掘り
  インタビューを行う。要件を掘り下げるとき、またはユーザーが
  "interview", "deep-dive", "discuss design" と発言したときに使う。
argument-hint: "[DESIGN.md path]"
---

# /interview - 深掘りインタビューコマンド

## 概要

DESIGN.md を読み込み、ユーザーと深掘りインタビューを行う。
インタビュー完了後、収集した仕様を DESIGN.md に書き込む。

## 使い方

```bash
/interview                    # デフォルト: docs/DESIGN.md
/interview path/to/DESIGN.md  # パスを指定
```

---

## 実行手順

### 1. DESIGN.md の読み込み

- `$ARGUMENTS` が指定されている場合: そのパスを使用
- それ以外: `docs/DESIGN.md` を使用

Read ツールでファイルを読み、内容を把握する。

### 2. インタビューの実施

DESIGN.md の内容に基づき、AskUserQuestion を用いて深掘りインタビューを行う。

**インタビューの観点**:
- 技術的な実装詳細
- UI/UX に関する考慮事項
- 懸念点とリスク
- トレードオフの判断
- エッジケースの扱い
- エラーハンドリング
- パフォーマンス要件
- セキュリティに関する考慮事項

**重要なルール**:
- **自明な質問はしない** - DESIGN.md に既に記載済み、または明らかに回答済みの事項はスキップする
- **同じ質問を繰り返さない** - 各ラウンドの前に、既にカバーした話題（DESIGN.md や過去の
  回答に含まれるもの）を洗い出し、除外する
- **深掘りする** - 表面的な確認ではなく、暗黙の前提や未決事項を掘り下げる
- **具体的な選択肢** - すべての質問は、各説明にトレードオフを明記した2〜4個の現実的な
  選択肢を提示する（ユーザーは常に「Other」を選べる）
- **バッチサイズ** - AskUserQuestion 1回あたり1〜2問（最大4問）
- **継続** - ユーザーが「done」と言うか、十分な情報が集まるまでインタビューを続ける

### インタビューの流れ

1. DESIGN.md の内容を確認する
2. 不明瞭な点や詳細が必要な箇所を特定する
3. AskUserQuestion で質問する（一度に1〜2問）
4. 回答を記録する
5. さらに質問が必要な場合は繰り返す
6. 十分な情報が集まったら終了する

### 質問例

```javascript
AskUserQuestion({
  questions: [
    {
      question: "[具体的な質問内容]",
      header: "Technical Details",
      options: [
        { label: "選択肢A", description: "説明A" },
        { label: "選択肢B", description: "説明B" }
      ],
      multiSelect: false
    }
  ]
})
```

### 3. 仕様の書き込み

インタビュー完了後、収集した情報を DESIGN.md に追記・更新する。

追記するセクション:

```markdown
## Specifications from Interview

### Technical Implementation
- [Collected details]

### UI/UX
- [Collected details]

### Concerns and Risks
- [Collected details]

### Decisions
- [Decisions made during interview]
```

---

## 終了条件

以下のいずれかに該当したら終了する:
- ユーザーが「done」「complete」「OK」などと回答した
- 十分な情報が集まり、これ以上質問がない
- 「特にありません」という回答を3回連続で受け取った

終了メッセージ:

```
✓ Interview completed

Updated file:
- [DESIGN.md path]

Appended sections:
- [List of appended section names]
```
