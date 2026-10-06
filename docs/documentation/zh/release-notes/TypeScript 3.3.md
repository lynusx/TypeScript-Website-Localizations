---
title: TypeScript 3.3
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-3-3.html
oneline: TypeScript 3.3 发布说明
---

## 调用联合类型时的改进

在 TypeScript 早期版本中，仅当可调用类型的联合具有完全相同的参数列表时，才_能够_被调用。

```ts
type Fruit = 'apple' | 'orange'
type Color = 'red' | 'orange'

type FruitEater = (fruit: Fruit) => number // eats and ranks the fruit
type ColorConsumer = (color: Color) => string // consumes and describes the colors

declare let f: FruitEater | ColorConsumer

// Cannot invoke an expression whose type lacks a call signature.
//   Type 'FruitEater | ColorConsumer' has no compatible call signatures.ts(2349)
f('orange')
```

然而在上述示例中，`FruitEater` 和 `ColorConsumer` 都应该能够接收字符串 `"orange"`，并返回 `number` 或 `string`。

在 TypeScript 3.3 中，这不再是一个错误。

```ts
type Fruit = 'apple' | 'orange'
type Color = 'red' | 'orange'

type FruitEater = (fruit: Fruit) => number // eats and ranks the fruit
type ColorConsumer = (color: Color) => string // consumes and describes the colors

declare let f: FruitEater | ColorConsumer

f('orange') // It works! Returns a 'number | string'.

f('apple') // error - Argument of type '"apple"' is not assignable to parameter of type '"orange"'.

f('red') // error - Argument of type '"red"' is not assignable to parameter of type '"orange"'.
```

在 TypeScript 3.3 中，这些签名的参数会被_交叉_（intersect）在一起，从而创建一个新签名。

在上面的示例中，参数 `fruit` 和 `color` 被交叉在一起，形成一个类型为 `Fruit & Color` 的新参数。
`Fruit & Color` 实际上与 `("apple" | "orange") & ("red" | "orange")` 相同，等价于 `("apple" & "red") | ("apple" & "orange") | ("orange" & "red") | ("orange" & "orange")`。
其中每一个不可能的交叉项都会归约为 `never`，最终只剩下 `"orange" & "orange"`，即 `"orange"`。

## 注意事项

该新行为仅在联合中至多有一个类型拥有多个重载，且至多有一个类型拥有泛型签名时才会生效。
这意味着像 `number[] | string[]` 上的 `map` 方法（它是泛型方法）仍然是不可调用的。

另一方面，像 `forEach` 这样的方法现在可以被调用了，但在 [`noImplicitAny`](/tsconfig#noImplicitAny) 下可能会遇到一些问题。

```ts
interface Dog {
  kind: 'dog'
  dogProp: any
}
interface Cat {
  kind: 'cat'
  catProp: any
}

const catOrDogArray: Dog[] | Cat[] = []

catOrDogArray.forEach((animal) => {
  //                ~~~~~~ error!
  // Parameter 'animal' implicitly has an 'any' type.
})
```

尽管如此，TypeScript 3.3 的能力显然更强了，并且添加显式类型注解即可正常工作。

```ts
interface Dog {
  kind: 'dog'
  dogProp: any
}
interface Cat {
  kind: 'cat'
  catProp: any
}

const catOrDogArray: Dog[] | Cat[] = []
catOrDogArray.forEach((animal: Dog | Cat) => {
  if (animal.kind === 'dog') {
    animal.dogProp
    // ...
  } else if (animal.kind === 'cat') {
    animal.catProp
    // ...
  }
})
```

## `--build --watch` 中复合项目的增量文件监视

TypeScript 3.0 引入了一项用于组织构建结构的新特性——“复合项目”（composite projects）。
其部分目标是确保用户可以将大型项目拆分为能够快速构建且保留项目结构的小型部分，同时不影响现有的 TypeScript 体验。
得益于复合项目，TypeScript 可以使用 `--build` 模式仅重新编译该组项目及其依赖项。
你可以将此理解为优化项目_间_（inter-project）的构建。

TypeScript 2.7 还通过全新的增量“构建器”（builder）API 引入了 `--watch` 模式构建。
类似地，其核心思想在于该模式仅重新检查并重新生成发生变更的文件，或者其依赖项可能影响类型检查的文件。
你可以将此理解为优化项目_内_（intra-project）的构建。

在 3.3 之前，使用 `--build --watch` 构建复合项目实际上并没有使用这种增量文件监视基础设施。
在 `--build --watch` 模式下，某个项目中的更新会强制对该项目进行完整构建，而不会判断该项目中有哪些文件受到了影响。

在 TypeScript 3.3 中，`--build` 模式的 `--watch` 标志_同样_利用了增量文件监视。
这意味着在 `--build --watch` 下的构建速度会显著提升。
在我们的测试中，相比原先的 `--build --watch` 时间，该功能使**构建时间减少了 50% 到 75%**。
[你可以阅读有关此项更改的原始 Pull Request](https://github.com/Microsoft/TypeScript/pull/29161) 以查看具体数字，但我们相信大多数复合项目用户都会在此获得显著提升。
