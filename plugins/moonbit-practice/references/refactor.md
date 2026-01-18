---
title: "MoonBit Refactoring Patterns"
---

# MoonBit リファクタリングパターン

## 何もしない処理を含む二分岐では match if のパターンマッチを優先する

```moonbit
let opt : Int? = Some(1)

///|
/// BAD
match opt {
  Some(v) => println("hello")
  None => ()
}

///|
/// Good
if opt is Some(v) {
  println("hello")
}
```

## guard によるアーリーリターンを優先する

条件を満たさない場合に早期に抜ける場合は `guard` を使う。

```moonbit
///|
/// BAD: ネストが深くなる
fn get_value(array : Array[Int], index : Int) -> Int? {
  if index >= 0 && index < array.length() {
    Some(array[index])
  } else {
    None
  }
}

///|
/// Good: guard でアーリーリターン
fn get_value(array : Array[Int], index : Int) -> Int? {
  guard index >= 0 && index < array.length() else { None }
  Some(array[index])
}
```

パターンマッチと組み合わせる:

```moonbit
///|
/// BAD: match でネストが深くなる
fn process(resources : Map[String, Resource], path : String) -> String raise Error {
  match resources.get(path) {
    Some(resource) => {
      match resource {
        PlainText(text) => process(text)
        _ => fail("\{path} is not plain text")
      }
    }
    None => fail("\{path} not found")
  }
}

///|
/// Good: guard is でフラットに
fn process(resources : Map[String, Resource], path : String) -> String raise Error {
  guard resources.get(path) is Some(resource) else { fail("\{path} not found") }
  guard resource is PlainText(text) else { fail("\{path} is not plain text") }
  process(text)
}
```

## 文字列操作のパフォーマンスを優先する際は StringView を使う

```moonbit
///|
/// BAD: String の連結は毎回新しい文字列を生成する
fn process_string(s : String) -> String {
  ...
}

///|
/// Good: StringView を使うとコピーを避けられる
fn process_string_view(s : StringView) -> Unit {
  ...
}
```

## C-style for より for-in を優先する

```moonbit
///|
/// BAD: C-style for
for i = 0; i < items.length(); i = i + 1 {
  println(items[i])
}

///|
/// Good: for-in
for item in items {
  println(item)
}

///|
/// Good: インデックスが必要な場合
for i, item in items {
  println("\{i}: \{item}")
}
```

## 単一式の関数はアロー関数を優先する

```moonbit
///|
/// BAD: 冗長
let f = fn(x) { x + 1 }
arr.map(fn(x) { x * 2 })

///|
/// Good: アロー関数
let f = fn { x => x + 1 }
arr.map(fn { x => x * 2 })
```

## for ループで値を返す場合は else を使う

```moonbit
///|
/// BAD: 変数を使って結果を保持
fn find_first(arr : Array[Int], target : Int) -> Int? {
  let mut result : Int? = None
  for i in arr {
    if i == target {
      result = Some(i)
      break
    }
  }
  result
}

///|
/// Good: for-else で直接返す
fn find_first(arr : Array[Int], target : Int) -> Int? {
  for i in arr {
    if i == target {
      break Some(i)
    }
  } else {
    None
  }
}
```
