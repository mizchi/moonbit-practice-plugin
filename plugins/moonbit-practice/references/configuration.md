---
title: "MoonBit Configuration Reference"
---

# MoonBit 設定リファレンス

## ファイル構造

```
my-project/
├── moon.mod.json    # モジュール設定（プロジェクト全体）
└── src/
    ├── moon.pkg       # パッケージ設定（新形式）
    └── main.mbt
```

## moon.pkg.json から moon.pkg への移行

`moon.pkg.json` は独自構文の `moon.pkg` に移行中。以下のコマンドで変換できる：

```bash
NEW_MOON_PKG=1 moon fmt
```

これにより `moon.pkg.json` が `moon.pkg` に変換される。

## moon.pkg（新形式）

JSON に比べてコメント対応、末尾カンマ許可、より簡潔な構文が特徴。

### インポート

```moonbit
// 基本インポート
import {
  "moonbitlang/async/io",
  "path/to/pkg" as @alias,
}

// テスト用インポート
import "test" {
  "path/to/pkg5",
}

// ホワイトボックステスト用インポート
import "wbtest" {
  "path/to/pkg7",
}
```

### オプション設定

```moonbit
options(
  "is-main": true,
  "bin-name": "name",
  link: { "native": { "cc": "gcc" } },
)
```

### JSON との比較

| 項目 | JSON 形式 | moon.pkg 形式 |
|------|-----------|---------------|
| コメント | ❌ 非対応 | ✅ 対応 |
| 末尾カンマ | ❌ 非対応 | ✅ 対応 |
| 可読性 | 低（冗長） | 高（簡潔） |

## moon.mod.json（モジュール設定）

### 必須フィールド

```json
{
  "name": "username/project-name",
  "version": "0.1.0"
}
```

### 依存関係

```json
{
  "deps": {
    "moonbitlang/x": "0.4.6",
    "username/other": { "path": "../other" }
  }
}
```

### メタ情報

```json
{
  "license": "MIT",
  "repository": "https://github.com/...",
  "description": "...",
  "keywords": ["example", "test"]
}
```

### ソースディレクトリ

```json
{
  "source": "src"
}
```

### ターゲット指定

```json
{
  "preferred-target": "js"
}
```

### 警告設定

```json
{
  "warn-list": "-2-4",
  "alert-list": "-alert_1"
}
```

## moon.pkg.json（パッケージ設定）

### メインパッケージ

```json
{
  "is-main": true
}
```

### 依存関係

```json
{
  "import": [
    "moonbitlang/quickcheck",
    { "path": "moonbitlang/x/encoding", "alias": "lib" }
  ],
  "test-import": [...],
  "wbtest-import": [...]
}
```

### 条件付きコンパイル

```json
{
  "targets": {
    "only_js.mbt": ["js"],
    "only_wasm.mbt": ["wasm"],
    "not_js.mbt": ["not", "js"],
    "debug_only.mbt": ["debug"],
    "js_release.mbt": ["and", ["js"], ["release"]]
  }
}
```

条件: `wasm`, `wasm-gc`, `js`, `debug`, `release`
演算子: `and`, `or`, `not`

### リンクオプション

#### JS バックエンド

```json
{
  "link": {
    "js": {
      "exports": ["hello", "foo:bar"],
      "format": "esm"
    }
  }
}
```

format: `esm`（デフォルト）, `cjs`, `iife`

#### Wasm バックエンド

```json
{
  "link": {
    "wasm-gc": {
      "exports": ["hello"],
      "use-js-builtin-string": true
    }
  }
}
```

### Pre-build（ビルド前処理）

```json
{
  "pre-build": [
    {
      "input": "a.txt",
      "output": "a.mbt",
      "command": ":embed -i $input -o $output"
    }
  ]
}
```

`:embed` でファイルを MoonBit ソースに変換（`--text` または `--binary`）

## 警告番号一覧

よく使うもの:
- `1` Unused function
- `2` Unused variable
- `11` Partial pattern matching
- `12` Unreachable code
- `27` Deprecated syntax

確認: `moonc build-package -warn-help`

## 参照

- Module: https://docs.moonbitlang.com/en/stable/toolchain/moon/module
- Package: https://docs.moonbitlang.com/en/stable/toolchain/moon/package
