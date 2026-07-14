# claude-plugins

[valbeat](https://github.com/valbeat) の個人用 [Claude Code](https://code.claude.com/docs) プラグインマーケットプレイス。スキル本文はすべて日本語で書かれている。

## インストール

```
/plugin marketplace add valbeat/claude-plugins
/plugin install writing@valbeat-plugins
```

CLI から:

```shell
claude plugin marketplace add valbeat/claude-plugins
claude plugin install writing@valbeat-plugins
```

## プラグイン一覧

| プラグイン | スキル | 説明 |
|---|---|---|
| `writing` | japanese-tech-writing, cognitive-rhythm-writing | 日本語技術文書の文章規範 — 整形・論証・認知リズム |
| `design` | design-guidelines, web-design-guidelines | 画像からのデザインガイドライン抽出、Web Interface Guidelines 準拠監査 |
| `git-workflow` | commit, pr, update-pr, fix-pr, issue, dependabot-merge, four-keys | Git/GitHub ワークフロー — コミット、PR、issue、dependabot マージ、DORA メトリクス。`git` と `gh` が必要 |
| `dev-workflow` | spec, impl, interview, dev, check | 仕様策定、TDD 実装、設計インタビュー、品質チェック |
| `skill-tools` | create-skill, claude-rule-update | メタスキル — スキル作成、CLAUDE.md ルール管理 |

スキルはプラグイン名前空間つきで呼び出す（例: `/git-workflow:commit`, `/writing:japanese-tech-writing`）。

## License

[MIT](LICENSE)
