---
name: fix-pr
allowed-tools: Read, Write, Edit, Bash(git:*), Bash(gh:*), Glob, Grep, AskUserQuestion
disable-model-invocation: true
argument-hint: "<pr-number>"
description: >-
  CI失敗、マージコンフリクト、レビューコメント（Copilotコードレビューを含む）など、
  GitHub PR の修正を包括的に扱う。PR がマージ可能になるまですべての問題を解決する。
  PR の修正、CI失敗の解決、レビューコメントの対応時、またはユーザーが
  "fix PR", "fix CI", "resolve PR issues", "PRを直して", "CIを直して"
  と言った場合に使う。
---

# Fix PR

## Context

- 現在のリポジトリ: !`gh repo view --json nameWithOwner -q .nameWithOwner 2>/dev/null || echo "Not a GitHub repository"`
- 現在のブランチ: !`git branch --show-current`
- Git status: !`git status --porcelain`
- プロジェクトの規約: @.claude/CLAUDE.md

## タスク

CI失敗、マージコンフリクト、レビューコメント（人間・Copilot両方）、その他 PR に関する問題を包括的に扱う。すべての問題が解決し、PR がマージ可能になるまで繰り返す。

## Steps

### Phase 1: PR の状態を把握する

1.  **PR の詳細、ステータス、マージ可否を確認する**
    ```bash
    gh pr view <pr-number>
    gh pr checks <pr-number>
    gh pr view <pr-number> --json mergeable,mergeStateStatus,baseRefName,headRefName
    ```

2.  **PR ブランチをチェックアウトする**
    ```bash
    gh pr checkout <pr-number>
    ```

### Phase 2: マージコンフリクトを解決する

3.  **コンフリクトを確認し解決する**
    ```bash
    # Determine base branch from PR
    BASE_BRANCH=$(gh pr view <pr-number> --json baseRefName -q .baseRefName)
    git fetch origin
    git merge origin/$BASE_BRANCH
    ```

    コンフリクトが発生した場合:
    - コンフリクトしている各ファイルを注意深く確認する
    - 解決する前に両側の意図を理解する
    - 両ブランチの正しいロジックを保ちながらコンフリクトを解決する
    - すべてのコンフリクト解決後:
      ```bash
      git add <resolved-files>
      git commit -m "fix: resolve merge conflicts with $BASE_BRANCH"
      ```
    - コンフリクトが複雑で自動解決できない場合は、AskUserQuestion で解決方針を確認する

### Phase 3: レビューコメントを処理する

4.  **すべてのレビュースレッドを取得する（Copilotを含む）**

    リポジトリの owner/repo を取得する:
    ```bash
    REPO_INFO=$(gh repo view --json owner,name -q '.owner.login + "/" + .name')
    OWNER=$(echo $REPO_INFO | cut -d/ -f1)
    REPO=$(echo $REPO_INFO | cut -d/ -f2)
    ```

    解決状況とともにすべてのレビュースレッドを取得する:
    ```bash
    gh api graphql \
      -F owner="$OWNER" \
      -F repo="$REPO" \
      -F prNumber=<pr-number> \
      -f query='
        query($owner: String!, $repo: String!, $prNumber: Int!) {
          repository(owner: $owner, name: $repo) {
            pullRequest(number: $prNumber) {
              reviewThreads(first: 100) {
                nodes {
                  id
                  isResolved
                  isOutdated
                  path
                  line
                  comments(first: 50) {
                    nodes {
                      id
                      databaseId
                      body
                      createdAt
                      author {
                        login
                      }
                    }
                  }
                }
              }
            }
          }
        }
      '
    ```

5.  **未解決の各レビュースレッドを処理する**

    未解決のスレッド（`isResolved: false`）のみに絞り込む。各スレッドについて:

    **a) コメント投稿者を特定する:**
    - Copilot: `author.login` に `copilot` を含む（例: `copilot-pull-request-reviewer`）
    - 人間のレビュアー: それ以外のログイン

    **b) コメントの種類を分類して対応する:**

    | 種類 | 対応 |
    |------|------|
    | **コード変更の指摘**（Copilot・人間問わず） | 指定されたファイル/行に修正を実装する |
    | **人間のレビュアーからの質問** | AskUserQuestion を使い、得た回答で返信する |
    | **軽微な指摘・スタイルの提案** | 単純なものは実装し、そうでなければユーザーに確認する |
    | **誤検知・不要な提案** | 対応するか却下するかユーザーに確認する |

    **c) 修正を実装した後、コメントに返信する:**
    スレッド内の最初（トップレベル）のコメントの `databaseId` を使う:
    ```bash
    gh api --method POST repos/$OWNER/$REPO/pulls/<pr-number>/comments/<databaseId>/replies \
      -f body="Fixed: <brief description of what was changed>"
    ```

    **d) スレッドを解決する:**
    Step 4 で取得したスレッドの `id`（GraphQL ノードID）を使う:
    ```bash
    gh api graphql \
      -F threadId="<thread-id>" \
      -f query='
        mutation($threadId: ID!) {
          resolveReviewThread(input: { threadId: $threadId }) {
            thread {
              id
              isResolved
            }
          }
        }
      '
    ```

    > **重要:** 次に進む前に、すべての未解決レビュースレッドを処理すること。`isOutdated: true` のスレッドは、まだ関係あるフィードバックを含む場合を除きスキップする。

### Phase 4: CI 失敗を修正する

6.  **CI 失敗の原因を分析する**
    ```bash
    gh pr checks <pr-number> --verbose
    ```
    失敗したチェックについて、詳細なログを取得する:
    ```bash
    gh run view <run-id> --log-failed
    ```
    よくある失敗パターンは [references/ci-failure-patterns.md](references/ci-failure-patterns.md) を参照。

7.  **修正を実装する**
    - エラーメッセージに基づいて修正する
    - 既存のコーディング規約に従う
    - 最小限の変更で解決する

8.  **最終的なローカル検証**
    > **重要:** コミット前に、CI パイプラインと同等のすべてのチェックを実行すること。
    > 正しいコマンドは `.github/workflows/`、`package.json` の scripts、`Makefile` を確認する。

### Phase 5: コミット・プッシュ・検証

9.  **すべての修正をコミットする**
    ```bash
    git add <changed-files>
    git commit -m "fix: resolve issues for PR #<pr-number>"
    ```

10. **プッシュして CI を監視する**
    ```bash
    git push
    gh pr checks <pr-number> --watch
    ```

11. **すべての問題が解決したことを検証する**
    ```bash
    gh pr checks <pr-number>
    gh pr view <pr-number> --json mergeable,mergeStateStatus
    ```

    レビュースレッドを再取得し、すべて解決済みであることを確認する:
    ```bash
    gh api graphql \
      -F owner="$OWNER" \
      -F repo="$REPO" \
      -F prNumber=<pr-number> \
      -f query='
        query($owner: String!, $repo: String!, $prNumber: Int!) {
          repository(owner: $owner, name: $repo) {
            pullRequest(number: $prNumber) {
              reviewThreads(first: 100) {
                nodes {
                  id
                  isResolved
                }
              }
            }
          }
        }
      ' | jq '[.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false)] | length'
    ```

    **未解決のスレッドや CI 失敗が残っている場合は、該当するフェーズに戻ってループする。**

12. **再レビューを依頼する（レビューコメントに対応した場合）**
    ```bash
    gh pr view <pr-number> --json reviews --jq '[.reviews[].author.login] | unique | .[]'
    gh pr edit <pr-number> --add-reviewer <reviewer1>,<reviewer2>
    ```

## Completion Criteria

以下のすべてが真であることを確認する（推測ではなくコマンドの出力で検証すること）まで、PR が修正済みだと宣言しないこと:

- [ ] `gh pr checks <pr-number>` がすべてのチェック通過（または修正をプッシュ済みで pending）を示している
- [ ] 未解決レビュースレッド数が 0（Step 11 のクエリが `0` を返す）
- [ ] `mergeable` が `MERGEABLE`（コンフリクトなし）
- [ ] すべての修正がプッシュ済み（`git status` がクリーンで、リモートと同期している）

いずれかの基準を満たせない場合（例: 外部承認が必要なチェックがある）は、成功したと主張せず、
どの基準が満たされておらずなぜかを正確に報告すること。

## Notes

- PR ブランチへの直接コミットは、PR 作成者の設定によっては制限される場合がある。その場合は新しいブランチを作成し、別の PR を開く。
- `--watch` オプションで CI の進行状況をリアルタイムに監視できる。
- 変更をプッシュする前に、必ずすべてのローカルチェックを通過させる。
- Copilot のレビューコメントは人間のレビューと同じ優先度で扱う — 妥当なものは修正を実装し、対応内容を説明して返信する。
- スレッドを解決する際は、レビュアーが対応内容を確認できるよう、解決する前に必ず返信すること。
