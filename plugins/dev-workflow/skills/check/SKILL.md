---
name: check
allowed-tools: Read, Bash
description: >-
  linter、formatter、build、test を含むプロジェクトの品質チェックを実行する。
  設定ファイル（package.json, go.mod, Cargo.toml, pyproject.toml）からプロジェクト
  種別を自動判定する。
  Trigger conditions: コード変更後の品質確認時、CI相当のローカルチェック実行時。
  Use when user says "check", "run tests", "lint", "format", "チェック", "テスト実行", or "品質確認".
argument-hint: "[--test|--lint|--format|--build|--all]"
---

# プロジェクトチェックの実行

## Context

- プロジェクト種別の検出: !`ls package.json Makefile pyproject.toml Cargo.toml go.mod 2>/dev/null || echo "No standard project files found"`
- 現在のディレクトリ: !`pwd`

## タスク

プロジェクトの品質チェックを実行する。プロジェクト種別を特定し、適切なチェックを実行する。

## 引数

- `--test`: テストのみ実行
- `--lint`: リンターのみ実行
- `--format`: フォーマッターのみ実行
- `--build`: ビルドのみ実行
- `--all` または引数なし: すべてのチェックを実行（format → lint → build → test）

## 手順

1. **設定ファイルからプロジェクト種別を検出する**:
   - `package.json` → Node.js
   - `go.mod` → Go
   - `Cargo.toml` → Rust
   - `pyproject.toml` / `setup.cfg` → Python
   - `Makefile` → Make ベース

2. **優先順位に従いコマンドを決定する**:
   1. プロジェクト定義のコマンドを最優先: `package.json` の scripts / `Makefile` の
      ターゲット / `.github/workflows/` で使われているコマンド
   2. 上記が存在しない場合のみ、下記の言語標準コマンドにフォールバックする

3. **チェックを順番に実行する**（`--all` の場合）:
   - フォーマッター（まず自動修正）
   - リンター（問題を検出）
   - ビルド（コンパイルチェック）
   - テスト（機能を検証）

## 言語別コマンド

### Go
```bash
# Format
gofmt -w .
goimports -w .  # if available

# Lint
go vet ./...
golangci-lint run  # if available
staticcheck ./...  # if available

# Build
go build ./...

# Test
go test ./...
go test -race ./...  # with race detection
go test -cover ./... # with coverage
```

### JavaScript/TypeScript (Node.js)
```bash
# Format
npm run format || npx prettier --write .

# Lint
npm run lint || npx eslint .

# Type Check
npm run typecheck || npx tsc --noEmit

# Build
npm run build

# Test
npm test
```

### Rust
```bash
# Format
cargo fmt

# Lint
cargo clippy -- -D warnings

# Build
cargo build

# Test
cargo test
```

### Python
```bash
# Format
black . || ruff format .

# Lint
ruff check . || flake8

# Type Check
mypy . || pyright

# Test
pytest
```

## 出力フォーマット

```
## Quality Check Results

### Format
✓ PASSED (or ✗ FAILED with details)

### Lint
✓ PASSED (or ✗ FAILED with details)

### Build
✓ PASSED (or ✗ FAILED with details)

### Test
✓ PASSED: X tests passed
(or ✗ FAILED: X passed, Y failed)

---
Overall: PASSED / FAILED
```

## /impl との連携

`/impl` から呼び出された場合:
- `--test` は RED/GREEN/REFACTOR サイクル中に頻繁に使用される
- フェーズ完了時にはフルチェック（`--all`）を実行する

## 注意事項

- リンターの前にフォーマッターを実行し、問題を自動修正する
- チェックが失敗した場合、失敗したコマンドと実際の出力をそのまま報告し停止する（`--all` モードの場合）
- 実際にコマンドを実行せずに PASSED と報告してはならない
- プロジェクト固有の指示は README を確認する
- CI との整合性のため、ローカルチェックがパイプラインと一致することを確認する
