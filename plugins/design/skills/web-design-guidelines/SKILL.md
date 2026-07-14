---
name: web-design-guidelines
description: UIコードをWeb Interface Guidelinesへの準拠観点でレビューする。Use when asked to "review my UI", "check accessibility", "audit design", "review UX", or "check my site against best practices".
argument-hint: <file-or-pattern>
---

# Web Interface Guidelines

Web Interface Guidelines への準拠状況に基づいてファイルをレビューする。

## 動作の仕組み

1. 下記の取得元URLから最新のガイドラインを取得する
2. 指定されたファイルを読み込む（ファイル/パターンが未指定の場合はユーザーに確認する）
3. 取得したガイドラインの全ルールと照合する
4. 簡潔な `file:line` 形式で所見を出力する

## ガイドラインの取得元

レビューのたびに最新のガイドラインを取得すること。

```
https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md
```

WebFetch を使って最新のルールを取得する。取得したコンテンツには全ルールと出力フォーマットの指示が含まれる。

## 使い方

ユーザーがファイルまたはパターンを引数として指定した場合:
1. 上記の取得元URLからガイドラインを取得する
2. 指定されたファイルを読み込む
3. 取得したガイドラインの全ルールを適用する
4. ガイドラインで指定された形式で所見を出力する

ファイルが指定されていない場合は、レビュー対象のファイルをユーザーに確認する。
