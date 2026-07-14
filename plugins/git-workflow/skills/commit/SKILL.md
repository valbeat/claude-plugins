---
name: commit
allowed-tools: Bash(git:*)
disable-model-invocation: true
argument-hint: "[message]"
description: >-
  Conventional Commits 仕様に従って git commit を作成する。変更のコミット、
  コミットメッセージの作成時、またはユーザーが "commit", "make a commit",
  "conventional commit" と言った場合に使う。
---

# Conventional Commit コマンド

## Context

- 現在のブランチ: !`git branch --show-current`
- 現在の変更: !`git status`
- ステージ済みの変更: !`git diff --cached --stat`
- 直近のコミット: !`git log --oneline -5`

## タスク

Conventional Commits 仕様に従ってコミットを作成する。現在の変更を確認し、適切なファイルをステージして、正しいフォーマットのコミットメッセージを作成する。

## Steps

1. **ブランチガード**: 現在のブランチが `main` または `master` の場合、先にフィーチャーブランチを作成する
   （`git switch -c <type>/<short-description>`）。ベースブランチに直接コミットしないこと。
2. `git status` と `git diff` を確認し、すべての変更を把握する
3. **1つの論理的な変更に関連するファイルだけをステージする**:
   - パスを指定して個別にファイルをステージする。`git add -A` や `git add .` は使わない
   - 差分に無関係な変更が含まれる場合は、別々のコミットに分割する（手順3〜4を繰り返す）
   - 認証情報、`.env` ファイル、生成物は絶対にステージしない
4. コミットを作成する: `git commit -m "<type>(<scope>): <subject>"`

## メッセージのルール

- Subject: 命令形、末尾にピリオドなし、50文字以内を目安にする
- Scope: 任意。ディレクトリ/モジュール名を使う（例: `auth`, `api`, `brew`）
- Body: subject だけでは「なぜ」が自明でない場合のみ追加する
- 言語: 英語

### Type の選択（判断テーブル）

| 変更内容 | Type |
|--------|------|
| ユーザー向けの新しい振る舞い | `feat` |
| 誤った振る舞いの修正 | `fix` |
| コード変更だが振る舞いは不変 | `refactor` |
| テストのみ | `test` |
| ドキュメントのみ | `docs` |
| フォーマット/空白のみ | `style` |
| パフォーマンス改善 | `perf` |
| ビルド、依存関係、設定、ツール | `chore` |

1つのコミットに複数の type が該当する場合は、優先順位で選ぶ: `feat` > `fix` > `refactor` > その他。

### 例

```bash
git commit -m "feat(auth): add OAuth2 login support"
```

Body 付き:
```bash
git commit -m "fix(api): handle null response from server

- Add null check before parsing response
- Return empty array instead of throwing error"
```

### Breaking Changes

コロンの前に `!` を付けるか、`BREAKING CHANGE:` フッターで示す:
```bash
git commit -m "feat(api)!: change response format

BREAKING CHANGE: response now returns array instead of object"
```

## 完了前に確認する

- [ ] `main`/`master` にコミットしていない
- [ ] `git status` に無関係なファイルが誤ってステージされていない
- [ ] Subject が `<type>(<scope>): <subject>` の形式で、英語で書かれている
