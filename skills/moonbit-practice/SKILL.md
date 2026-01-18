---
name: moonbit-practice
description: MoonBit code generation best practices. Use when writing MoonBit code to avoid common AI mistakes with syntax, tests, and benchmarks.
---

# MoonBit Practice Guide

AI が MoonBit コードを生成する際のベストプラクティス。
ここにある内容がわからなかった時 `moonbit-docs` スキルを使って公式ドキュメントを検索する

## やってほしいこと

### コードナビゲーション: Read/Grep より moon ide を優先

MoonBit プロジェクトでは **Read ツールや grep より `moon ide` コマンドを優先** すること。

```bash
# ❌ 避ける: ファイルを直接読む
Read src/parser.mbt

# ✅ 推奨: シンボルから定義を探す
moon ide peek-def Parser::parse
moon ide goto-definition -tags 'pub fn' -query 'parse'

# ❌ 避ける: grep で検索
grep -r "fn parse" .

# ✅ 推奨: セマンティック検索
moon ide find-references parse
moon ide outline src/parser.mbt
```

**理由:**
- `moon ide` はセマンティック検索（定義とコールサイトを区別）
- grep はコメントや文字列も拾う
- `moon doc` で API を素早く把握できる

### その他のルール

- `moon doc '<Type>'` で API を調べてから実装する
- moon.pkg.json / moon.mod.json を編集する前に reference/configuration.md を確認
- CLAUDE.md を更新するときは reference/agents.md を確認

## Common Pitfalls（よくあるミス）

- **変数・関数名に大文字を使わない** - コンパイルエラー
- **`mut` はフィールド変更ではなく再代入時のみ** - Array の push には不要
- **`return` は不要** - 最後の式が戻り値
- **メソッドには `Type::` プレフィックス必須**
- **`++` `--` は非対応** - `i = i + 1` または `i += 1`
- **エラー伝播に `try` 不要** - 自動伝播（Swift と異なる）
- **`await` キーワードなし** - async 関数は `async fn` で宣言するだけ
- **C-style for より range for を優先** - `for i in 0..<n {...}`
- **レガシー構文**: `function_name!(...)` や `function_name(...)?` は非推奨

## AI がよく間違える文法

### 型引数の位置

```moonbit
///| NG: fn identity[T] は古い構文
fn identity[T](val: T) -> T { val }

///| OK: 型引数は fn の直後
fn[T] identity(val: T) -> T { val }
```

### raise 構文

```moonbit
///|
/// NG: -> T!Error は削除された
fn parse(s: String) -> Int!Error { ... }

///|
/// OK: raise キーワードを使う
fn parse(s: String) -> Int raise Error { ... }
```

`Int raise` と省略すると、`Int raise Error` になる。
async fn ではデフォルトで `raise` 相当になり `noraise` を使う場合、エラーが発生しないことを強制する。`try~catch` を強制する。

### マクロ呼び出し

```moonbit
///|
/// NG: ! 付きは削除された
assert_true!(true)

///|
/// OK
assert_true(true)
```

### 複数行テキスト

```moonbit
let text =
  #|line 1
  #|line 2
```

### コメントと分割ブロック

`///|` は分割ブロック。`///` コメントは直下の `///|` ブロックに紐づく。

```moonbit
///|
/// この関数は foo です
fn foo() -> Unit { ... }

///|
/// この関数は bar です
fn bar() -> Unit { ... }
```

ファイル冒頭に `///|` を連続して書くと別ブロックとして分割されるので避ける。

## スナップショットテスト

`moon test -u` で `inspect(val)` の `content=""` が自動更新される。

```moonbit
test "snapshot" {
  inspect([1, 2, 3], content="")  // moon test -u で自動補完
}
```

実行後:

```moonbit
test "snapshot" {
  inspect([1, 2, 3], content="[1, 2, 3]")
}
```

## Doc Test

`.mbt.md` ファイルまたは `///|` インラインコメント内で使用可能。

| コードブロック | 動作 |
|---------------|------|
| ` ```mbt check ` | LSP で検査される |
| ` ```mbt test ` | `test {...}` 相当として実行 |
| ` ```moonbit ` | 何も実行されない（表示のみ） |

例（インラインコメント）:

```moonbit

///|
/// 整数を1増やす
/// ```mbt test
/// inspect(incr(41), content="42")
/// ```
pub fn incr(x : Int) -> Int {
  x + 1
}
```

## リリース前の準備

リリース前に以下を実行:

```bash
moon fmt   # コードフォーマット
moon info  # 型定義ファイル生成
```

`pkg.generated.mbti` は型定義ファイルであり、`moon info` で自動生成される。これを直接編集しない。

## ビルトイン型のメソッドを調べる

```bash
moon doc StringView   # StringView のメソッド一覧
moon doc Array        # Array のメソッド一覧
moon doc Map          # Map のメソッド一覧
```

## クイックリファレンス

| トピック | コマンド | 詳細 |
|---------|---------|------|
| テスト | `moon test` | https://docs.moonbitlang.com/en/stable/language/tests |
| スナップショット更新 | `moon test -u` | 同上 |
| フィルタ付きテスト | `moon test --filter 'glob'` | 特定テストのみ実行 |
| ベンチマーク | `moon bench` | https://docs.moonbitlang.com/en/stable/language/benchmarks |
| Doc Test | `moon check` / `moon test` | https://docs.moonbitlang.com/en/stable/language/docs |
| フォーマット | `moon fmt` | - |
| 型定義生成 | `moon info` | - |
| ドキュメント参照 | `moon doc <Type>` | - |

## moon ide ツール

grep より正確なコードナビゲーション。詳細は `reference/ide.md` 参照。

```bash
# シンボル定義を表示
moon ide peek-def Parser::read_u32_leb128

# パッケージのアウトライン
moon ide outline .

# 参照箇所を検索
moon ide find-references TranslationUnit

# 型定義にジャンプ（位置指定）
moon ide peek-def Parser -loc src/parse.mbt:46:4
```

## Functional for loop

可能な限り functional for loop を使う。読みやすく、推論しやすい。

```moonbit
// 状態を持つ functional for loop
for i = 0, sum = 0; i <= 10; {
  continue i + 1, sum + i  // 状態の更新
} else {
  sum  // ループ終了時の値
}

// range for（推奨）
for i in 0..<n { ... }
for i, v in array { ... }  // index と value
```

## Error Handling

MoonBit は checked errors を使用。詳細は `reference/ffi.md` 参照。

```moonbit
///| エラー型の宣言
suberror ParseError {
  InvalidEof
  InvalidChar(Char)
}

///| raise で宣言、自動伝播
fn parse(s: String) -> Int raise ParseError {
  if s.is_empty() { raise ParseError::InvalidEof }
  ...
}

///| Result に変換
let result : Result[Int, ParseError] = try? parse(s)

///| try-catch で処理
parse(s) catch {
  ParseError::InvalidEof => -1
  _ => 0
}
```

## Assets

assets/ci.yaml は CI 用の GitHub Actions ワークフロー定義
