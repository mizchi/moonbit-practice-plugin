---
title: "MoonBit FFI Reference"
---

# MoonBit FFI リファレンス

## 外部型の宣言

```moonbit
#external
type ExternalRef
```

- Wasm: `externref`
- JS: `any`
- C: `void*`

## 外部関数の宣言

### JavaScript バックエンド

```moonbit
///| モジュール.関数 形式
fn cos(d : Double) -> Double = "Math" "cos"

///| インライン JavaScript
extern "js" fn cos(d : Double) -> Double =
  #|(d) => Math.cos(d)

///| 複数行
extern "js" fn fetch_json(url : String) -> Value =
  #|(url) => fetch(url).then(r => r.json())
```

### Wasm バックエンド

```moonbit
///| ホストからインポート
fn cos(d : Double) -> Double = "math" "cos"

///| インライン Wasm
extern "wasm" fn identity(d : Double) -> Double =
  #|(func (param f64) (result f64))
```

### C バックエンド

```moonbit
extern "C" fn put_char(ch : UInt) = "putchar"
```

## 型マッピング

### JavaScript

| MoonBit | JavaScript |
|---------|-----------|
| `String` | `string` |
| `Bool` | `boolean` |
| `Int`, `Double` | `number` |
| `BigInt` | `bigint` |
| `Bytes` | `Uint8Array` |
| `Array[T]` | `T[]` |
| `#external type` | `any` |

### Wasm

| MoonBit | Wasm |
|---------|------|
| `Bool`, `Int` | `i32` |
| `Int64` | `i64` |
| `Float` | `f32` |
| `Double` | `f64` |
| `#external type` | `externref` |

## JavaScript FFI パターン

### undefined/null の扱い

```moonbit
#external
pub type Value

extern "js" fn Value::undefined() -> Value = "() => undefined"
extern "js" fn Value::null() -> Value = "() => null"
extern "js" fn Value::is_undefined(self : Value) -> Bool =
  #|(n) => Object.is(n, undefined)
```

### 型キャスト（%identity）

```moonbit
fn[T] Value::cast_from(value : T) -> Value = "%identity"
fn[T] Value::cast(self : Value) -> T = "%identity"
```

### エラーハンドリング

```moonbit
extern "js" fn wrap_ffi(
  op : () -> Value,
  on_ok : (Value) -> Unit,
  on_error : (Value) -> Unit,
) -> Unit =
  #|(op, on_ok, on_error) => {
  #|  try { on_ok(op()); }
  #|  catch (e) { on_error(e); }
  #|}
```

### Node.js モジュールの読み込み

```moonbit
extern "js" fn require_ffi(path : String) -> Value =
  #|(path) => require(path)

// 使用例
let path_module = require_ffi("node:path")
```

### #module による直接インポート（推奨）

```moonbit
#module("node:fs")
extern "js" fn readFileSync(path : String, encoding : String) -> String = "readFileSync"

#module("node:path")
extern "js" fn basename(path : String) -> String = "basename"
extern "js" fn dirname(path : String) -> String = "dirname"
```

`#module` 属性を使うと、指定したモジュールから直接関数をインポートできる。

## 関数のエクスポート

`moon.pkg.json` で設定:

```json
{
  "link": {
    "js": {
      "exports": ["add", "fib:fibonacci"],
      "format": "esm"
    }
  }
}
```

## コールバック

### FuncRef（クロージャなし）

```moonbit
///| 自由変数を持たない関数のみ
fn register(callback : FuncRef[() -> Unit]) -> Unit
```

### 通常のクロージャ

JavaScript では自動的にクロージャとして扱われる。

## 定数 enum のカスタマイズ

```moonbit
enum Flags {
  Read = 1
  Write = 2
  ReadWrite = 3
}
```

C ライブラリのフラグとの互換性に便利。

## 参照

- FFI: https://docs.moonbitlang.com/en/stable/language/ffi
- JS FFI Guide: https://www.moonbitlang.com/pearls/moonbit-jsffi
