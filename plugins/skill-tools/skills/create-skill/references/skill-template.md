# スキルテンプレート

## 最小構成テンプレート

```markdown
---
name: skill-name
description: >-
  [何をするか]。ユーザーが "[トリガー1]", "[トリガー2]",
  "[トリガー3]" と言った場合に使う。
allowed-tools: Read, Glob, Grep
---

## Your Task

[このスキルが何をするかの明確な説明]

## Steps

1. [最初のステップ]
2. [次のステップ]
3. [3番目のステップ]

## Example

User: "[入力例]"
Result: [期待される出力の説明]

## Error Handling

- [条件]の場合は、[回復のためのアクション]を行う
```

## 参照ファイル付きテンプレート

```markdown
---
name: skill-name
description: >-
  [何をするか]。[いつ使うか]。ユーザーが "[トリガー1]",
  "[トリガー2]", "[トリガー3]" と言った場合に使う。
allowed-tools: Read, Bash(specific-command:*), Glob, Grep
argument-hint: [想定される引数の説明]
---

## Context

- 関連する状態: !`command-to-check-state`
- 既存のリソース: !`ls relevant/directory/`

## Your Task

[このスキルが何をするかの明確な説明]

[トピック]の詳細なガイダンスは @references/topic-guide.md を参照

## Steps

1. **[フェーズ名]**: [説明]
   - 詳細A
   - 詳細B

2. **[フェーズ名]**: [説明]
   - 詳細は @references/detailed-guide.md を参照

3. **[フェーズ名]**: [説明]

## Example

User: "/skill-name some-argument"
Result:
- [出力1]
- [出力2]

## Error Handling

- [よくあるエラー]の場合は、[回復方法]
- [エッジケース]の場合は、[フォールバック]

## Success Criteria

- [ ] [測定可能な成果1]
- [ ] [測定可能な成果2]
```

## Frontmatter フィールド一覧

### 必須

| フィールド | 説明 |
|-------|-------------|
| `name` | スキル識別子。フォルダ名と一致させる（kebab-case） |
| `description` | WHAT + WHEN + トリガー（最大1024文字、XML不可） |

### 任意

| フィールド | 説明 |
|-------|-------------|
| `allowed-tools` | 許可するツールのカンマ区切りリスト |
| `argument-hint` | 想定される引数の説明（ヘルプに表示される） |
| `model` | 希望するモデル: `sonnet`, `opus`, `haiku` のいずれか |
| `user-invocable` | `false` にするとユーザーからは呼び出せず Claude のみが呼び出せる |
| `disable-model-invocation` | `true` にするとユーザーのみが呼び出せる |
| `context` | `fork` にすると独立したサブエージェントのコンテキストで実行される |
