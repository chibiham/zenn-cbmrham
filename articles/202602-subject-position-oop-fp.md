---
title: "主語の位置：OOPと関数型における関数適用スタイルの違い"
emoji: "🔄"
type: "tech"
topics: ["typescript", "oop", "関数型プログラミング", "設計", "プログラミングパラダイム"]
published: false
---

:::message
この記事はAIを使用して執筆しています。
:::

## 導入：「shapeに対してcalcを行う」という感覚

コードを書くとき、無意識に「主語」を置いている。

```typescript
// OOP的
shape.area()      // 「shapeが」areaを返す

// FP的
area(shape)       // 「area関数が」shapeを処理する
```

どちらも同じ計算をしているのに、読み方が違う。前者は「shapeに対してareaを行う」、後者は「areaがshapeを使う」。

この違いは単なる構文の好みではなく、**データと操作の関係性をどう捉えるか**というパラダイムの違いを反映している。

## レシーバーと引数：知識の所在

OOPでは、データ（オブジェクト）が振る舞いを「持っている」。

```typescript
class Circle {
  constructor(private radius: number) {}
  area(): number {
    return Math.PI * this.radius ** 2
  }
}

const circle = new Circle(5)
circle.area()  // circleは自分のareaを知っている
```

関数型では、関数がデータを「知っている」。

```typescript
type Circle = { radius: number }

const area = (c: Circle): number => Math.PI * c.radius ** 2

area(circle)  // area関数がCircleの構造を知っている
```

| 観点 | OOP | FP |
|------|-----|-----|
| 知識の所在 | データが振る舞いを持つ | 関数がデータを知る |
| 拡張の方向 | 新しい種類のデータを追加しやすい | 新しい操作を追加しやすい |
| 凝集の単位 | クラス（データ+操作） | モジュール（型+関数群） |

これがExpression Problemの本質でもある。

## Expression Problem：拡張のジレンマ

上の表で示した「拡張の方向」の違いは、**Expression Problem**として知られる古典的な問題に対応している。

### 問題の本質

「既存のコードを変更せずに、データ型と操作の両方を拡張できるか？」という問いに対し、OOPとFPは異なる答えを持つ。

```typescript
// OOP：新しいデータ型の追加は簡単
class Triangle extends Shape {
  area(): number { ... }  // 既存コードを変更せずTriangleを追加
}

// しかし、新しい操作の追加は困難
// → 全クラスにperimeter()を追加する必要がある
```

```typescript
// FP：新しい操作の追加は簡単
const perimeter = (s: Shape): number => ...  // 新しい関数を追加するだけ

// しかし、新しいデータ型の追加は困難
// → 全ての関数でTriangleのケースを追加する必要がある
```

### どちらを選ぶか

これは「どちらが優れているか」ではなく、**ドメインの変化の方向を予測する設計判断**。

| 予想される変化 | 適したスタイル |
|--------------|--------------|
| 新しい種類のデータが増える | OOP（クラス追加） |
| 新しい操作・処理が増える | FP（関数追加） |
| 両方が同程度に増える | Visitor, 拡張関数等の折衷案 |

ECサイトで「商品の種類」が増えるならOOP的に、「分析・レポート機能」が増えるならFP的に設計すると変更が楽になる。

## パイプ演算子：FPでもデータ起点に書く

関数型言語の多くは、**パイプ演算子**でデータ起点の記述を可能にしている。

```elixir
# Elixir
shape |> area() |> format()
```

```fsharp
// F#
shape |> area |> format
```

```haskell
-- Haskell（Data.Functionの&演算子）
shape & area & format
```

これらは `shape.area().format()` とほぼ同じ読み方ができる。「shapeに対してareaを行い、その結果に対してformatを行う」。

### 参考：Haskellのスタイル

| 記法 | 方向 | 用途 |
|------|------|------|
| `format (area shape)` | 内→外 | 素朴な適用 |
| `format $ area $ shape` | 右→左 | 括弧省略 |
| `shape & area & format` | 左→右 | データフロー |
| `format . area` | 右→左 | 関数合成 |

Haskellコミュニティでは `.` と `$` が主流で、「データを流す」より「関数を組み立てる」文化が強い。

## Kotlin：拡張関数という中間解

Kotlinはパイプ演算子を持たないが、**拡張関数**で両方の良さを取り入れている。

```kotlin
// 拡張関数：クラス外で定義するが、メソッド構文で呼べる
fun Shape.area(): Double = when (this) {
    is Circle -> PI * radius * radius
    is Rectangle -> width * height
}

shape.area()  // OOP的な構文
```

さらに**スコープ関数**でパイプ的な記述も可能：

```kotlin
shape
    .let { calculateArea(it) }
    .let { formatResult(it) }
```

拡張関数の特徴：
- 構文は `shape.area()` （OOP的、レシーバーがある）
- 定義はクラス外（FP的、後から操作を追加できる）
- Expression Problemの「操作追加」側を楽にしつつ、レシーバー構文を維持

## TypeScript：両スタイルを選べる

TypeScriptにはパイプ演算子がない（TC39でStage 2提案中）が、ライブラリで同様のことができる。

### 型と関数を同じファイルで管理

```typescript
// domain/shape.ts
type Shape =
  | { type: "circle"; radius: number }
  | { type: "rectangle"; width: number; height: number }

const area = (s: Shape): number =>
  s.type === "circle"
    ? Math.PI * s.radius ** 2
    : s.width * s.height

const format = (n: number): string => `${n.toFixed(2)}㎡`

export { Shape, area, format }
```

これは実質的に**モジュール = 型 + 操作のセット**という凝集を実現している。OOPのクラスと同じ構造を、判別共用体と関数の組み合わせで作る。

### データ起点（左から右）

```typescript
import { pipe } from "fp-ts/function"

pipe(shape, area, format)
// shape → area → format
// 「shapeに対してareaして、formatする」
```

### 関数合成（右から左の定義、後で適用）

```typescript
import { flow } from "fp-ts/function"

const formatArea = flow(area, format)
// area → format を合成した新しい関数
formatArea(shape)
```

### 素朴な入れ子

```typescript
format(area(shape))
// 内側から外側にデータが流れる
```

| スタイル | 書き方 | 向いている場面 |
|---------|--------|--------------|
| `pipe(shape, area, format)` | データが流れる | 1回の変換処理 |
| `flow(area, format)` | 関数を組み立てる | 再利用する変換 |
| `format(area(shape))` | 数学的 | シンプルな合成 |

## 思考の起点：データか操作か

「shapeに対してareaを行う」が自然に感じるか、「areaがshapeを処理する」が自然に感じるかは、**思考の起点**の違い。

### データ起点の思考

- 「このデータに対して何ができるか？」
- IDEで `shape.` と打つと操作一覧が出る
- OOP的、Kotlinの拡張関数、TypeScriptのpipe

### 操作起点の思考

- 「この操作は何に対して使えるか？」
- 関数の型シグネチャ `Shape -> number` を見る
- FP的、Haskellの関数合成

どちらが正しいというものではなく、**問題領域や個人の思考スタイルによって使い分ける**もの。

## 実用的な指針

TypeScriptで判別共用体を使う場合の実用的なアプローチ：

1. **型と関数を同じファイルに配置**して凝集を高める
2. **変換が1回きり**なら `pipe` でデータ起点に書く
3. **変換を再利用する**なら `flow` で関数を合成する
4. **シンプルな場合**は素朴な入れ子で十分

```typescript
// domain/order.ts
type Order = Draft | Paid | Shipped

const ship = (o: Paid): Shipped => { ... }
const cancel = (o: Draft | Paid): Cancelled => { ... }
const toSummary = (o: Order): string => { ... }

export { Order, ship, cancel, toSummary }
```

このファイル構成は、「Order型に対して何ができるか」を一箇所に集約している。構文上は関数だが、概念的にはOOPのクラスと同じ凝集度を持つ。

## 結論：構文を超えた設計の一貫性

OOPか関数型かという二項対立ではなく、**どちらの構文を使っても同じ設計原則が適用できる**ことが重要。

- 凝集度：関連するデータと操作を近くに置く
- 結合度：モジュール間の依存を最小化する
- Expression Problem：拡張の方向を意識して設計する

「主語の位置」は構文上の選択であり、その下にある設計思想——データと操作の関係性、変更の波及範囲、モジュールの境界——は共通している。

自分の思考スタイルに合った構文を選びつつ、設計の一貫性を保つこと。それがパラダイムを超えた実践的なアプローチだと考える。
