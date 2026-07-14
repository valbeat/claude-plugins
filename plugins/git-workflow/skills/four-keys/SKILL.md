---
allowed-tools: Bash(gh:*), Bash(git log:*), Bash(git tag:*), Bash(git diff:*), Bash(git rev-list:*), Bash(git show:*), Bash(date:*), Bash(bc:*), Bash(sort:*), Bash(wc:*), Bash(awk:*), Bash(head:*), Bash(tail:*), Bash(grep:*), Bash(uniq:*), Bash(jq:*), Read
argument-hint: "[--period 30d|90d|180d|1y] [--repo owner/repo] [--deploy-tag-pattern 'v*'] [--deploy-workflow 'deploy']"
name: four-keys
disable-model-invocation: true
context: fork
description: >-
  デプロイ頻度、変更のリードタイム、変更失敗率、MTTR を含む Four Keys（DORA メトリクス）
  を計測する。DevOps パフォーマンスの計測時、またはユーザーが "four keys",
  "DORA metrics", "deployment frequency" と言った場合に使う。
model: sonnet
---

# Four Keys（DORAメトリクス）計測

## Context

- 現在のリポジトリ: !`gh repo view --json nameWithOwner -q .nameWithOwner 2>/dev/null || echo "Not a GitHub repository"`
- デフォルトブランチ: !`gh repo view --json defaultBranchRef -q .defaultBranchRef.name 2>/dev/null || echo "unknown"`
- GitHub CLI アクセス: 必須

## 概要

GitHub リポジトリから4つの DORA (DevOps Research and Assessment) メトリクスを計測する:

1. **Deployment Frequency (DF)** - 本番環境へのデプロイ頻度
2. **Lead Time for Changes (LT)** - 最初のコミットから本番デプロイまでの時間
3. **Change Failure Rate (CFR)** - デプロイのうち障害を引き起こした割合
4. **Mean Time to Recovery (MTTR)** - 障害からの復旧にかかる時間

## Arguments

| 引数 | デフォルト | 説明 |
|----------|---------|-------------|
| `--period` | `90d` | 分析期間: `30d`, `90d`, `180d`, `1y` |
| `--repo` | (現在のリポジトリ) | 対象リポジトリ（`owner/repo`） |
| `--deploy-tag-pattern` | `v*` | デプロイを識別するタグの glob パターン |
| `--deploy-workflow` | (なし) | デプロイに使う GitHub Actions のワークフロー名 |
| `--deploy-branch` | デフォルトブランチ | デプロイを追跡するブランチ |
| `--failure-labels` | `bug,incident,hotfix` | デプロイ障害を示す Issue/PR ラベル |

## 計測プロセス

### Step 1: 分析期間を決定する

`--period` 引数（デフォルト: 90d）をパースし、開始日を計算する。

```bash
# 開始日を計算する
START_DATE=$(date -v-90d +%Y-%m-%dT00:00:00Z 2>/dev/null || date -d "90 days ago" --iso-8601=seconds 2>/dev/null)
```

### Step 2: デプロイを特定する

以下の戦略を優先順位順に試す:

#### Strategy A: GitHub Actions のワークフロー実行（`--deploy-workflow` 指定時）
```bash
gh run list --workflow "$WORKFLOW" --status completed --branch "$BRANCH" \
  --json createdAt,conclusion,headSha,displayTitle \
  --limit 200 \
  --jq "[.[] | select(.createdAt >= \"$START_DATE\")]"
```

#### Strategy B: GitHub Releases
```bash
gh release list --limit 200 \
  --json tagName,createdAt,isPrerelease \
  --jq "[.[] | select(.isPrerelease == false and .createdAt >= \"$START_DATE\")]"
```

#### Strategy C: パターンに一致する Git タグ
```bash
git tag --sort=-creatordate --format='%(creatordate:iso-strict) %(refname:short)' \
  | while read date tag; do
    if [[ "$tag" == $PATTERN ]] && [[ "$date" > "$START_DATE" ]]; then
      echo "$date $tag"
    fi
  done
```

どの戦略を使い、いくつのデプロイが見つかったかを報告する。

### Step 3: デプロイ頻度（DF）

以下を計算する:
- 期間内のデプロイ合計数
- 日次/週次/月次のデプロイ数
- パフォーマンスレベルを分類する

**パフォーマンスレベル:**
| レベル | 頻度 |
|-------|-----------|
| Elite | オンデマンド（1日に複数回） |
| High | 1日1回〜週1回の間 |
| Medium | 週1回〜月1回の間 |
| Low | 月1回未満 |

### Step 4: 変更のリードタイム（LT）

各デプロイについて、含まれるコミットを特定し、最初のコミットからデプロイまでの時間を計算する:

```bash
# 連続するデプロイ（タグ/リリース）のペアごとに
git log --format="%H %aI" "$PREV_TAG..$CURRENT_TAG" --first-parent
# リードタイム = デプロイのタイムスタンプ - 最も早いコミットのタイムスタンプ
```

以下を計算する:
- リードタイムの中央値
- P50, P90 のリードタイム
- パフォーマンスレベルを分類する

**パフォーマンスレベル:**
| レベル | リードタイム |
|-------|-----------|
| Elite | 1時間未満 |
| High | 1日〜1週間の間 |
| Medium | 1週間〜1ヶ月の間 |
| Low | 1ヶ月超 |

### Step 5: 変更失敗率（CFR）

以下を確認して障害を特定する:

1. デプロイ後の**revert コミット**
```bash
git log --grep="^Revert" --grep="^revert" --format="%H %aI %s" --since="$START_DATE"
```

2. **hotfix デプロイ**（名前に "hotfix", "fix", "patch" を含むタグ/リリース）

3. **障害ラベル付きの Issue/PR**
```bash
gh issue list --state closed --label "bug,incident,hotfix" \
  --json number,title,createdAt,closedAt \
  --jq "[.[] | select(.createdAt >= \"$START_DATE\")]" \
  --limit 200
```

以下を計算する:
- 障害を引き起こしたデプロイ数 / デプロイ合計数
- パフォーマンスレベルを分類する

**パフォーマンスレベル:**
| レベル | CFR |
|-------|-----|
| Elite | 0-5% |
| High | 5-10% |
| Medium | 10-15% |
| Low | 16-30%+ |

### Step 6: 平均修復時間（MTTR）

特定した障害について、復旧時間を計算する:

1. 障害イベントを、その解決（次の成功デプロイ、issue クローズ、fix PR マージ）と対応付ける
2. 障害検知から復旧までの時間を計算する

```bash
# 障害ラベル付きのissueについて
gh issue list --state closed --label "bug,incident" \
  --json createdAt,closedAt \
  --jq "[.[] | select(.createdAt >= \"$START_DATE\") | {created: .createdAt, closed: .closedAt}]"
```

**パフォーマンスレベル:**
| レベル | MTTR |
|-------|------|
| Elite | 1時間未満 |
| High | 1日未満 |
| Medium | 1日〜1週間の間 |
| Low | 1週間超 |

## 出力フォーマット

```markdown
# Four Keys Report

**Repository:** owner/repo
**Period:** YYYY-MM-DD ~ YYYY-MM-DD (Xd)
**Deployment Detection:** [strategy used]

## Summary

| Metric | Value | Level |
|--------|-------|-------|
| Deployment Frequency | X.X / week | ⭐ Elite |
| Lead Time for Changes | Xh XXm (median) | 🟢 High |
| Change Failure Rate | X.X% | 🟡 Medium |
| Mean Time to Recovery | Xh XXm | 🟢 High |

**Overall DORA Level: [Elite/High/Medium/Low]**

## Deployment Frequency

- Total deployments: X
- Per day: X.X
- Per week: X.X
- Per month: X.X
- Trend: [↑ Increasing / → Stable / ↓ Decreasing]

[Weekly deployment chart if applicable]

## Lead Time for Changes

- Median: Xh XXm
- P50: Xh XXm
- P90: Xd Xh
- Shortest: XXm
- Longest: Xd Xh

## Change Failure Rate

- Failed deployments: X / Y total (X.X%)
- Failure types:
  - Reverts: X
  - Hotfixes: X
  - Incidents: X

## Mean Time to Recovery

- Mean: Xh XXm
- Median: Xh XXm
- Fastest: XXm
- Slowest: Xd Xh
- Incidents tracked: X

## Recommendations

[Based on metrics, provide actionable recommendations to improve each metric]
```

## パフォーマンスレベルのアイコン

- ⭐ Elite
- 🟢 High
- 🟡 Medium
- 🔴 Low

## 注意事項

- 対象リポジトリへのアクセス権を持つ `gh` CLI の認証が必要
- 精度は一貫したデプロイ運用（タグ、リリース、ワークフロー）に依存する
- Change Failure Rate は revert、hotfix タグ、ラベル付き issue から近似的に算出する
- MTTR は障害ラベルが使われている場合、issue/incident のライフサイクルから計算する
- 最良の結果を得るには、リリース/タグの命名規則を一貫させ、インシデントに適切にラベルを付けること
- どのデプロイ検出戦略でも結果が得られない場合は、その制約を報告し、設定改善を提案する

## 例

```bash
# 現在のリポジトリを直近90日間で計測（デフォルト）
/four-keys

# 特定のリポジトリを直近180日間で計測
/four-keys --repo facebook/react --period 180d

# ワークフローベースのデプロイ検出を使う
/four-keys --deploy-workflow "production-deploy"

# カスタムのタグパターンと期間
/four-keys --deploy-tag-pattern "release-*" --period 1y

# カスタムの障害ラベル
/four-keys --failure-labels "bug,outage,p0"
```
