# agent-plugins

[valbeat](https://github.com/valbeat) の個人用プラグインマーケットプレイス。[Claude Code](https://code.claude.com/docs) 向けに書いており、Codex も `.claude-plugin/marketplace.json` を読んで同じプラグインを入れられる。スキル本文はすべて日本語で書かれている。

旧名 `claude-plugins`（2026-09-17 に改名。マーケットプレイス名 `valbeat-plugins` とプラグイン ID は変えていない）。

## Codex で使う

Codex は `.agents/plugins/marketplace.json` を `.claude-plugin/marketplace.json` より優先して読む。Codex 用カタログには、Claude Code 固有の仕組み（サブエージェント、モデル指定、AskUserQuestion 前提の対話、`~/.claude` の構成）に依存しないプラグインだけを載せる。

| プラグイン | Codex | 理由 |
|---|---|---|
| writing / design | 載せる | 文章・デザインの規範だけで、実行環境に依存しない |
| git-workflow | 載せる | git / gh の手順が中心。fix-pr の AskUserQuestion は Codex では通常の質問になる |
| dev-workflow | 載せない | spec / impl / dev がサブエージェントとモデル指定（Fable 等）を前提にしている |
| skill-tools | 載せない | Claude Code のスキル（`~/.claude/skills`）と CLAUDE.md を扱う |

```shell
codex plugin marketplace add valbeat/agent-plugins
codex plugin add git-workflow@valbeat-plugins
```

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
