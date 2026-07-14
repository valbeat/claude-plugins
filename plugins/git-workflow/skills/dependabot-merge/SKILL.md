---
name: dependabot-merge
allowed-tools: Bash(gh:*)
disable-model-invocation: true
description: >-
  すべてのオープンな Dependabot PR を自動レビューし、auto-merge を有効化する。
  単一リポジトリ、org 全体、複数リポジトリのいずれの処理にも対応する。
  依存関係更新のマージ時、またはユーザーが "merge dependabot",
  "auto-merge deps", "handle dependabot PRs", "org全体のdependabot" と言った場合に使う。
argument-hint: "[--dry-run] [--include-major] [--org <name>] [--repo <owner/name>]"
---

# Dependabot 自動マージ

すべてのオープンな Dependabot PR を自動レビューし、auto-merge を有効化する。
単一リポジトリ、org 全体、特定リポジトリへの絞り込みに対応する。

## Context

- **Repository**: 現在の git リポジトリ、特定のリポジトリ、または org 内のすべてのリポジトリ
- **Target**: `app/dependabot` が作成した PR
- **State**: オープンな PR のみ

## Arguments

- `--dry-run`: プレビューモード。実際には変更を行わず、処理対象を表示する。
- `--include-major`: メジャーバージョン更新も対象に含める（通常は安全のためスキップする）。
- `--org <name>`: 指定した GitHub organization 内の全リポジトリを処理する。
- `--repo <owner/name>`: 特定のリポジトリを処理する（複数指定可）。

## スコープの解決

引数に基づき対象スコープを決定する。

### 単一リポジトリ（デフォルト）
`--org` も `--repo` も指定されない場合。現在の git リポジトリを使用する。

### Org 全体（`--org <name>`）
org 内の非アーカイブリポジトリを取得し、それぞれを処理する:
```bash
gh repo list <org> --no-archived --source --json nameWithOwner --limit 500 -q '.[].nameWithOwner'
```
- アーカイブ済みリポジトリは `--no-archived` により**常に除外**される
- フォークは `--source` により除外される（Dependabot はデフォルトでフォーク上では動かない）
- オープンな Dependabot PR があるリポジトリのみ処理する（PR が0件のリポジトリはスキップ）

### 特定リポジトリ（`--repo <owner/name>`）
指定したリポジトリのみを処理する。`--repo` フラグは複数指定できる。

## Steps

1. **前提条件の確認**
   - `gh` CLI が認証済みであることを確認する: `gh auth status`
   - `--org` も `--repo` も指定がない場合: カレントディレクトリが git リポジトリであることを確認する

2. **対象リポジトリの解決**
   - **デフォルト**: 現在のリポジトリのみ
   - **`--org`**: リポジトリ一覧を取得する（上記「スコープの解決」参照）
   - **`--repo`**: 指定されたリポジトリをそのまま使う
   - 処理前に対象リポジトリの数をログ出力する

3. **各対象リポジトリについて、オープンな Dependabot PR を取得する**
   ```bash
   # 単一リポジトリ（カレントディレクトリ）
   gh pr list --author app/dependabot --state open --json number,title,url,headRefName

   # 特定リポジトリまたは組織全体での反復処理
   gh pr list --repo <owner/name> --author app/dependabot --state open --json number,title,url,headRefName
   ```
   - オープンな Dependabot PR が0件のリポジトリはスキップする（スキップしたリポジトリの出力は不要）

4. **各 PR について更新タイプを判定する**
   - PR タイトルをパースしてバージョン変更を検出する
   - `patch`, `minor`, `major` のいずれかに分類する（下記「バージョン検出」参照）

5. **メジャー更新はスキップする**（`--include-major` 指定時を除く）
   - メジャー更新は破壊的変更を含む可能性がある
   - スキップした PR は手動レビュー用にログ出力する

6. **PR の diff を取得する**
   ```bash
   # 現在のリポジトリ
   gh pr diff <number>
   # 特定リポジトリ
   gh pr diff <number> --repo <owner/name>
   ```

7. **変更内容をレビューする**
   - diff を分析し、何が変更されたかを理解する
   - 不審な変更や予期しない変更がないか確認する
   - 標準的な依存関係更新であることを検証する

8. **レビューコメントを投稿する**（dry-run 時はスキップ）
   ```bash
   gh pr comment <number> -b "<review comment>" [--repo <owner/name>]
   ```

9. **PR を承認する**（dry-run 時はスキップ）
   ```bash
   gh pr review <number> --approve -b "LGTM - automated review by Claude" [--repo <owner/name>]
   ```

10. **Auto-Merge を有効化する**（dry-run 時はスキップ）
    ```bash
    gh pr merge <number> --auto --merge [--repo <owner/name>]
    ```

11. **サマリーを出力する**
    - 処理したすべての PR をリポジトリ別にテーブル表示する（org/複数リポジトリモードの場合）
    - リポジトリごとと全体の統計（合計、処理済み、スキップ、失敗）を表示する

## バージョン検出

Dependabot の PR タイトルは以下のパターンに従う:
- `Bump <package> from <old_version> to <new_version>`
- `Update <package> requirement from <old_version> to <new_version>`

バージョン変更の分類:
- **Major**: 最初の数字が変わる（例: 1.x.x → 2.x.x）
- **Minor**: 2番目の数字が変わる（例: 1.1.x → 1.2.x）
- **Patch**: 3番目の数字が変わる（例: 1.1.1 → 1.1.2）

安全のためのルール（major として扱いデフォルトでスキップする）:
- **0.x バージョン**: 0.x でのマイナーバンプ（例: 0.3.x → 0.4.0）は semver の慣習上
  破壊的変更となりうる — `major` として分類する
- **グループ化された更新**（1つの PR で複数パッケージを更新）: PR 内の全パッケージのうち
  最も大きいバンプで分類する
- **パースできないバージョン**（タイトルに明確な old→new の semver がない）: `major` として
  分類し、手動レビュー用にログ出力する — 推測しない

## レビューコメントテンプレート

レビューコメントはすべて英語で書く:

```markdown
## Automated Dependency Update Review

### Summary
- **Package**: {package_name}
- **Version Change**: {old_version} → {new_version}
- **Update Type**: {patch|minor|major}

### Review Notes
{analysis_of_changes}

### Decision
This update has been reviewed and approved for auto-merge.

---
*Reviewed by Claude Code*
```

## 出力フォーマット

### 単一リポジトリモード

```markdown
## Dependabot Auto-Merge Summary

### Processed PRs
| PR# | Package | Update | Status |
|-----|---------|--------|--------|
| #123 | lodash | 4.17.20 → 4.17.21 (patch) | ✅ Auto-merge enabled |
| #124 | axios | 0.21.0 → 1.0.0 (major) | ⏭️ Skipped (major) |
| #125 | react | 17.0.1 → 17.0.2 (patch) | ❌ Failed |

### Statistics
- **Total PRs**: X
- **Processed**: Y
- **Skipped (major)**: Z
- **Failed**: W
```

### Org / 複数リポジトリモード

結果をリポジトリ別にグループ化する:

```markdown
## Dependabot Auto-Merge Summary — org: <org-name>

### <owner/repo-a> (3 PRs)
| PR# | Package | Update | Status |
|-----|---------|--------|--------|
| #10 | lodash | 4.17.20 → 4.17.21 (patch) | ✅ Auto-merge enabled |
| #11 | webpack | 4.x → 5.x (major) | ⏭️ Skipped (major) |
| #12 | eslint | 8.50.0 → 8.51.0 (minor) | ✅ Auto-merge enabled |

### <owner/repo-b> (1 PR)
| PR# | Package | Update | Status |
|-----|---------|--------|--------|
| #5 | typescript | 5.2.0 → 5.3.0 (minor) | ✅ Auto-merge enabled |

### Overall Statistics
- **Repos scanned**: N
- **Repos with Dependabot PRs**: M
- **Total PRs**: X
- **Processed**: Y
- **Skipped (major)**: Z
- **Failed**: W
```

## 安全に関する注意

1. **メジャー更新はデフォルトでスキップする** — 破壊的変更を含む可能性があり、手動レビューとテストが必要になるため。

2. **承認前に必ず diff をレビューする**。以下を確認する:
   - lockfile 以外での予期しないファイル変更
   - 不審なコード変更
   - 設定ファイルの変更

3. **Auto-merge にはリポジトリ設定が必要**:
   - リポジトリ設定で auto-merge が有効になっている必要がある
   - ブランチ保護ルールでステータスチェックの通過が必須の場合がある

4. **まず dry-run で確認する**: 慣れないリポジトリでは必ず `--dry-run` を使い、処理対象を事前に確認する。

## エラー処理

- PR を処理できない場合、エラーをログに残し、次の PR の処理を続ける
- 最終サマリーですべての失敗を報告する
- よくある問題:
  - マージコンフリクト（PR に rebase が必要）
  - ステータスチェックの失敗
  - 権限不足
