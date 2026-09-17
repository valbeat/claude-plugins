# agent-plugins

[valbeat](https://github.com/valbeat) の個人用プラグインマーケットプレイス。[Claude Code](https://code.claude.com/docs) 向けに書いており、Codex も `.claude-plugin/marketplace.json` を読んで同じプラグインを入れられる。スキル本文はすべて日本語で書かれている。

旧名 `claude-plugins`（2026-09-17 に改名。マーケットプレイス名 `valbeat-plugins` とプラグイン ID は変えていない）。

## インストール

```
/plugin marketplace add valbeat/agent-plugins
/plugin install writing@valbeat-plugins
```

CLI から:

```shell
claude plugin marketplace add valbeat/agent-plugins
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
