# MoonBit Practice Guide

AI が MoonBit コードを生成する際のベストプラクティス。
ここにある内容がわからなかった時 `moonbit-docs` スキルを使って公式ドキュメントを検索する

## やってほしいこと

- grep の前に `moon ide goto-definition -tags 'pub fn' -query 'maximum'` を積極的に使って型を追跡する
- moon.pkg.json moon.mod.json の設定を書き換える前に references/configuration.md を確認する
- Moonbit に関する CLAUDE.md を更新するときは `references/agents.md` を確認すること

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
| CI | `moon test -u` | |
| ベンチマーク | `moon bench` | https://docs.moonbitlang.com/en/stable/language/benchmarks |
| Doc Test | `moon check` / `moon test` | https://docs.moonbitlang.com/en/stable/language/docs |
| フォーマット | `moon fmt` | - |
| 型定義生成 | `moon info` | - |
| ドキュメント参照 | `moon doc <Type>` | - |


## Assets

assets/ci.yaml は CI 用の GitHub Actions ワークフロー定義
