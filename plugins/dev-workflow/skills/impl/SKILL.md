---
name: impl
description: >-
  厳格な RED→GREEN→REFACTOR サイクルによる TDD 手法に従って、機能実装や
  バグ修正を行う。機能を実装するとき、TDD でバグを修正するとき、または
  ユーザーが "implement", "build feature", "TDD" と発言したときに使う。
argument-hint: "[task description]"
---

# /impl - TDD 開発コマンド

このコマンドは Kent Beck の TDD 手法に従う。
テストファーストのアプローチで RED→GREEN→REFACTOR サイクルを厳格に遵守する。

**統合コマンド**: 内部で `/check` と `/review` を使用する。

## 使い方

```bash
/impl Add validation to login form
/impl  # 対話形式で開始
```

---

## [1/4] タスク準備

### タスク説明の取得

- `$ARGUMENTS` が存在する場合: そのまま使用
- 空の場合: AskUserQuestion で確認

### ブランチガード

このコマンドは TDD サイクル中にコミットを行う。開始前に現在のブランチを確認し、
`main`/`master` 上であれば先に feature ブランチを作成する
（`git switch -c feat/<short-description>` または `fix/<short-description>`）。
ベースブランチへの直接コミットは絶対に行わない。

### 既存ドキュメントの確認

Read ツールで `docs/DESIGN.md` と `docs/TODO.md` の存在を確認する。

**docs/TODO.md が存在する場合**:
- 内容を読み、フェーズ構造を把握する
- 各フェーズの RED/GREEN/REFACTOR タスクを確認する

TODO.md 構造の例:
```markdown
### Phase 1: Implement version calculation

- [ ] [RED] Write test for calculate_latest
- [ ] [GREEN] Implement calculate_latest
- [ ] [REFACTOR] Refactor calculate_latest
```

---

## [2/4] フェーズ実行

### TodoWrite によるタスク管理

現在のフェーズのタスクを TodoWrite で管理する。

### RED-GREEN-REFACTOR サイクル

#### RED（テストを書く）

1. TodoWrite を `in_progress` に更新
2. 期待される入出力に基づきテストを書く
3. `/check --test` を実行し、**失敗を確認する**
4. **テストが正しい理由で失敗しているか検証する**:
   - 正しい: アサーション失敗（期待値と実際値の不一致）
   - 誤り: コンパイルエラー、import エラー、構文エラー、fixture 不足
   - 誤った理由で失敗している場合は、テストのセットアップを修正し、
     アサーション失敗になるまで再実行する
5. 失敗した状態でコミットする（テストが正しいことの証明）。コミットメッセージ: `test: <what the test verifies>`
6. TodoWrite を `completed` に更新

#### GREEN（実装する）

1. TodoWrite を `in_progress` に更新
2. テストをパスさせる**最小限の実装**を書く
3. `/check --test` を実行し、**成功を確認する**
4. **テストをパスさせるためにテスト自体を修正してはならない。** テスト自体が誤って
   いるように見える場合は、変更前に一旦停止しユーザーに確認する
5. 3回修正を試みてもテストが失敗し続ける場合は、一旦停止し失敗内容をユーザーに報告する
6. TodoWrite を `completed` に更新

#### REFACTOR

1. TodoWrite を `in_progress` に更新
2. 設計原則に従いコード品質を改善する
3. `/check --test` を実行し、**成功を維持する**（テストは変更しない）
4. リファクタリングすべき点がなければ、その旨を明示して次に進む（変更をでっち上げない）
5. TodoWrite を `completed` に更新

**設計原則チェックリスト**:

| カテゴリ | チェック項目 |
|----------|-------------|
| SOLID | 単一責任、依存性逆転 |
| テスト容易性 | 依存性注入、純粋関数 |
| 構造 | 高凝集・低結合、DRY |
| シンプルさ | YAGNI、KISS |

---

## [3/4] フェーズ承認

フェーズ内の RED/GREEN/REFACTOR タスクをすべて完了したら:

### Step 1: セルフレビュー

`/review --uncommitted --brief` を実行する:

```
Review changed files.
Report Critical/Warning issues only.
```

**問題が見つかった場合**:
1. 問題を修正する
2. `/check --test` で成功を確認する
3. セルフレビューを再実行する
4. 問題がなくなるまで繰り返す（最大3ラウンド — 3ラウンド経過後も問題が残る場合は、
   残存する問題をユーザーに報告し対応を確認する）

### Step 2: 品質チェック

`/check` を実行する:

```bash
# すべてのチェックを実行 (lint, format, build, test)
```

**失敗した場合**:
1. 問題を修正する
2. `/check` を再実行する
3. パスするまで繰り返す

### Step 3: フェーズ完了

AskUserQuestion で承認を得る:

```
Phase X completed.

Implemented:
- [Features implemented]

Changed files:
- [File list]

Test results: X tests passed
Review results: PASSED

Proceed to next phase?
```

---

## [4/4] 完了

### 完了サマリー

```
✓ Development completed

Implemented:
- [List of features/fixes]

Created/Modified files:
- [File path list]

Test results:
- X tests passed

Quality checks:
- lint: PASSED
- format: PASSED
- build: PASSED
```

### 次のアクション

AskUserQuestion で確認する:
- **Commit**: `/commit` を実行する
- **Done**: 開発を終了する

---

## TDD の絶対原則

1. **テストなしでコードを書かない**
2. **RED→GREEN→REFACTOR サイクルに従う**
3. **最小限の実装** - 現在のテストをパスさせる分だけコードを書く
4. **GREEN のときのみリファクタリングする**
5. **RED の状態でコミットする** - テストが正しいことの証明になる

## アンチパターン

- テストを「後で」書く
- テストを書く前に実装する
- RED の状態でリファクタリングする
- 複数フェーズを同時に実装する
- 品質チェックをスキップする
