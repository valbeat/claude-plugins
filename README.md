# claude-plugins

Personal [Claude Code](https://code.claude.com/docs) plugin marketplace by [valbeat](https://github.com/valbeat).

## Install

```
/plugin marketplace add valbeat/claude-plugins
/plugin install writing@valbeat-plugins
```

Or from the CLI:

```shell
claude plugin marketplace add valbeat/claude-plugins
claude plugin install writing@valbeat-plugins
```

## Plugins

| Plugin | Skills | Description |
|---|---|---|
| `writing` | japanese-tech-writing, cognitive-rhythm-writing | Japanese technical writing standards — style, rigor, and cognitive rhythm. Skill content is written in Japanese. |
| `design` | design-guidelines, web-design-guidelines | Extract design guidelines from images; audit UI against web interface guidelines |
| `git-workflow` | commit, pr, update-pr, fix-pr, issue, dependabot-merge, four-keys | Git/GitHub workflows — commits, PRs, issues, dependabot merges, DORA metrics. Requires `git` and `gh`. |
| `dev-workflow` | spec, impl, interview, dev, check | Spec planning, TDD implementation, design interviews, quality checks |
| `skill-tools` | create-skill, claude-rule-update | Meta skills — create new skills, maintain CLAUDE.md rules |

Skills are invoked with the plugin namespace, e.g. `/git-workflow:commit`, `/writing:japanese-tech-writing`.

## License

[MIT](LICENSE)
