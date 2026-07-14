---
name: create-skill
description: >-
  新しい Claude スキルを作成するための対話型ガイド。ユースケースの定義、
  frontmatter の生成、指示文の作成、検証まで順を追って進める。ユーザーが
  "create a skill", "build a new skill", "help me make a skill", "new skill",
  "add a skill" と言った場合に使う。
allowed-tools: Read, Bash(ls:*), Glob, Grep
---

## Context

- 現在のプロジェクトスキル: !`ls -la .claude/skills/ 2>/dev/null || echo "No project skills"`
- 利用可能なユーザースキル: !`ls -la ~/.claude/skills/ 2>/dev/null || echo "No user skills"`
- プロジェクトのガイドライン: !`head -50 .claude/CLAUDE.md 2>/dev/null || echo "No CLAUDE.md found"`

## タスク

スキル作成の専門家として、ユーザーが構造化された Claude スキルを作成できるよう、ステップごとに案内する。各ステップを完了させてから次に進むこと。

## Step 1: 既存スキルの調査

何かを作成する前に、既にあるものを調査する。

1. `.claude/skills/` と `~/.claude/skills/` のスキルを一覧する
2. 類似のスキルを読み、確立されたパターンを把握する
3. ツールの使い方、引数の扱い、構造といった慣習をメモする

## Step 2: 目的の把握

ユーザーに以下を尋ねる。
1. このスキルはどんな問題を解決するか？
2. 誰が、いつ使うか？
3. 期待される出力は何か？

その上で以下を決定する。
- **アプローチ**: Problem-first（ユーザーのニーズから出発）か Tool-first（利用可能なツール/MCPから出発）か
- **カテゴリ**: Document & Asset Creation / Workflow Automation / MCP Enhancement
- **パターン**: Sequential workflow / Multi-MCP coordination / Iterative refinement / Context-aware tool selection / Domain-specific intelligence
- **配置場所**: プロジェクトスキル（`.claude/skills/`）かユーザースキル（`~/.claude/skills/`）か

詳細は @references/skill-categories-and-patterns.md を参照。

## Step 3: description の記述

WHAT + WHEN + トリガーの公式で description を書く。

```
[何をするか] + [いつ使うか] + [トリガーフレーズ]
```

要件:
- **常に三人称で書く**（"I can help you" ではなく "Processes files" のように）
- 1024文字以内
- XML の山括弧を含めない
- 自然なトリガーフレーズを3〜5個含める
- 100以上のスキルの中から Claude が選べるだけの具体性を持たせる

例やベストプラクティスは @references/description-writing-guide.md を参照。

## Step 4: スキルの生成

適切なテンプレートを使ってスキルディレクトリと SKILL.md を作成する。

必須の frontmatter フィールド:
- `name`: フォルダ名と一致させる（kebab-case、最大64文字、予約語不可）
- `description`: Step 3 で作成したもの

任意フィールド: `allowed-tools`, `argument-hint`, `model`, `user-invocable`, `disable-model-invocation`, `context`

### 作成の原則

- **簡潔さが要**: Claude がまだ知らない文脈だけを含める。各段落について「Claude にこれが本当に必要か？」を問う
- **SKILL.md は500行以内**: プログレッシブディスクロージャーを使い、必要に応じて読み込む参照ファイルに分割する
- **参照は1階層まで**: すべての参照ファイルは SKILL.md から直接リンクする（ネストした参照チェーンは作らない）
- **適切な自由度**: 内容の壊れやすさに応じて具体性を変える（壊れやすい操作には正確なスクリプトを、柔軟なタスクには一般的なガイダンスを）
- **用語の一貫性**: 用語を1つに統一し、一貫して使う
- **モデルにロバストであること**: 小さいモデル（Sonnet/Haiku）でも品質が落ちないようにスキルを書く
  - 曖昧な判断（「適切に生成する」など）を、判断テーブル・優先順位・具体的なルールに置き換える
  - スキルが起動するサブエージェントには、オーケストレーターが展開する必要がある1行の説明ではなく、完全なプロンプトテンプレートを埋め込む
  - 破壊的な操作（PR/issue本文の編集、ファイルの上書きなど）では、save → modify → verify → apply の手順を明記する。本文の受け渡しはインラインでの引用ではなく `--body-file` やヒアドキュメントを使う
  - すべてのループに反復回数の上限とエスカレーション経路（「3回失敗したらユーザーに報告する」など）を設ける
  - 複数ステップのワークフローの最後には、コマンドで検証可能な完了チェックリストを置く
  - 最終出力にプレースホルダー（`[TBD]`）を残すことを明示的に禁止する

テンプレートとフィールド一覧は @references/skill-template.md を参照。

## Step 5: 成功基準の定義

測定可能な成功基準を定義する。
- トリガー精度: 適切なフレーズでスキルが起動するか？
- ワークフローの完遂: 期待される出力を生成するか？
- エラー処理: よくある失敗から適切に回復できるか？

## Step 6: 検証

スキルが完成したとみなす前に、検証チェックリストを一通り確認する。

チェックリスト全文は @references/validation-checklist.md を参照。

問題が発生した場合は @references/troubleshooting.md を参照。

セキュリティ上の考慮事項は @references/security-restrictions.md を参照。

## セッション例

User: "I need a skill to run database migrations"

**Step 1** — 類似パターンがないか既存スキルを確認する。

**Step 2** — 質問:
- どのデータベースシステムか？どの migration ツールか？
- ロールバックに対応する必要があるか？複数環境か？

分類: Workflow Automation、Sequential workflow パターン、プロジェクトスキル。

**Step 3** — description:
```yaml
description: >-
  Execute database migrations with environment selection, dry-run support,
  and rollback capability. Use when user says "run migrations",
  "migrate database", "rollback migration", or "check pending migrations".
```

**Step 4** — 適切な frontmatter とステップを含む `.claude/skills/run-migrations/SKILL.md` を生成する。

**Step 5** — 成功基準: migration が正しく実行される、ロールバックが機能する、dry-run が実行せずに変更内容を表示する。

**Step 6** — 検証チェックリストを一通り確認する。

## 出力サマリー

すべてのステップ完了後、以下をまとめる。

1. **作成したスキル**: 配置場所、名前、カテゴリ、パターン
2. **作成したリソース**: 補助ファイル、参照ファイル
3. **使い方**: `/skill-name` と呼び出し例
4. **次のステップ**: 実際の会話でスキルをテストし、フィードバックをもとに改善する
