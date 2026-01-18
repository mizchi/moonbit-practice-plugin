# MoonBit Practice Plugin

MoonBit コード生成のベストプラクティスを提供する Claude Code プラグインです。

## インストール

```bash
claude plugin marketplace add mizchi/moonbit-practice-plugin
claude plugin install moonbit-practice@moonbit-plugins
```

## 使い方

```bash
/moonbit-practice
```

## 含まれるリファレンス

- **ide.md**: `moon ide` を使ったコードナビゲーション
- **ffi.md**: MoonBit FFI リファレンス
- **refactor.md**: リファクタリングパターン
- **configuration.md**: moon.mod.json / moon.pkg.json 設定
- **agents.md**: プロジェクトレイアウトと開発ワークフロー

## ディレクトリ構造

```
moonbit-practice-plugin/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   └── moonbit-practice/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       ├── commands/
│       │   └── moonbit-practice.md
│       ├── references/
│       │   ├── agents.md
│       │   ├── configuration.md
│       │   ├── ffi.md
│       │   ├── ide.md
│       │   └── refactor.md
│       └── assets/
│           └── ci.yaml
└── README.md
```

## License

MIT
