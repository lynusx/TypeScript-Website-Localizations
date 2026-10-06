---
title: TypeScript 2.3
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-2-3.html
oneline: TypeScript 2.3 发布说明
---

## ES5/ES3 的生成器与迭代器

_首先了解一些 ES2016 术语：_

##### 迭代器

[ES2015 引入了 `Iterator`](http://www.ecma-international.org/ecma-262/6.0/#sec-iteration)，它是一个暴露了 `next`、`return` 和 `throw` 三个方法的对象，其接口定义如下：

```ts
interface Iterator<T> {
  next(value?: any): IteratorResult<T>
  return?(value?: any): IteratorResult<T>
  throw?(e?: any): IteratorResult<T>
}
```

这种迭代器适用于遍历同步可用的值，例如数组元素或 Map 的键。若一个对象拥有返回 `Iterator` 对象的 `Symbol.iterator` 方法，则称该对象为"可迭代的"（iterable）。

迭代器协议还定义了部分 ES2015 特性的目标对象，例如 `for..of`、展开运算符，以及解构赋值中的数组剩余元素。

##### 生成器

[ES2015 还引入了"生成器"](http://www.ecma-international.org/ecma-262/6.0/#sec-generatorfunction-objects)，它是一类可以通过 `Iterator` 接口和 `yield` 关键字产出中间计算结果的函数。生成器还可以通过 `yield *` 在内部将调用委托给另一个可迭代对象。例如：

```ts
function* f() {
  yield 1
  yield* [2, 3]
}
```

##### 新增 `--downlevelIteration`

此前，仅当编译目标为 ES6/ES2015 或更高版本时才支持生成器。此外，对于低于 ES6/ES2015 的编译目标，操作迭代器协议的构造（如 `for..of`）也只支持数组类型。

TypeScript 2.3 通过 [`downlevelIteration`](/tsconfig#downlevelIteration) 标志，为 ES3 和 ES5 编译目标新增了对生成器及迭代器协议的完整支持。

启用 [`downlevelIteration`](/tsconfig#downlevelIteration) 后，编译器会使用新的类型检查和代码生成逻辑：若被迭代对象上存在 `[Symbol.iterator]()` 方法则调用它，否则在该对象上创建一个合成的数组迭代器。

> 请注意，对于非数组值，这要求运行时环境原生支持 `Symbol.iterator` 或提供相应的 polyfill。

在使用 [`downlevelIteration`](/tsconfig#downlevelIteration) 时，`for..of` 语句、数组解构以及数组、函数调用和 new 表达式中的展开元素均可在 ES5/ES3 中使用 `Symbol.iterator`（若可用）；即便数组在运行时或设计时未定义 `Symbol.iterator`，也可以对数组使用上述特性。

## 异步迭代

TypeScript 2.3 新增了对 [当前 TC39 提案](https://github.com/tc39/proposal-async-iteration) 所描述的异步迭代器和异步生成器的支持。

##### 异步迭代器

异步迭代引入了 `AsyncIterator`，其与 `Iterator` 类似，区别在于 `AsyncIterator` 的 `next`、`return` 和 `throw` 方法返回的是包含迭代结果的 `Promise`，而非迭代结果本身。这使得调用方可以订阅 `AsyncIterator` 推进到产出新值这一异步通知。`AsyncIterator` 的结构如下：

```ts
interface AsyncIterator<T> {
  next(value?: any): Promise<IteratorResult<T>>
  return?(value?: any): Promise<IteratorResult<T>>
  throw?(e?: any): Promise<IteratorResult<T>>
}
```

若一个对象拥有返回 `AsyncIterator` 对象的 `Symbol.asyncIterator` 方法，则称该对象支持异步迭代。

##### 异步生成器

[异步迭代提案](https://github.com/tc39/proposal-async-iteration) 还引入了"异步生成器"，即同样可以产出中间计算结果的 async 函数。异步生成器也可以通过 `yield*` 将调用委托给可迭代对象或异步可迭代对象：

```ts
async function* g() {
  yield 1
  await sleep(100)
  yield* [2, 3]
  yield* (async function* () {
    await sleep(100)
    yield 4
  })()
}
```

与普通生成器一样，异步生成器只能是函数声明、函数表达式，或类/对象字面量的方法，不能是箭头函数。异步生成器除了需要有效的全局 `Promise` 实现（原生或 ES2015 兼容的 polyfill）外，还需要有效的 `Symbol.asyncIterator` 引用（原生 symbol 或 shim）。

##### `for-await-of` 语句

ES2015 引入了 `for..of` 语句用于遍历可迭代对象。类似地，异步迭代提案引入了 `for..await..of` 语句用于遍历异步可迭代对象：

```ts
async function f() {
  for await (const x of g()) {
    console.log(x)
  }
}
```

`for..await..of` 语句只能在异步函数或异步生成器中使用。

##### 注意事项

- 请注意，我们对异步迭代器的支持依赖于运行时存在 `Symbol.asyncIterator`。你可能需要为其提供 polyfill，简单的做法如：`(Symbol as any).asyncIterator = Symbol.asyncIterator || Symbol.for("Symbol.asyncIterator");`
- 你还需要在 [`lib`](/tsconfig#lib) 选项中包含 `esnext`，以获取 `AsyncIterator` 的声明（若尚未包含）。
- 最后，如果你的编译目标是 ES5 或 ES3，还需要设置 `--downlevelIterators` 标志。

## 泛型参数默认值

TypeScript 2.3 新增了对泛型类型参数声明默认值的支持。

##### 示例

假设有一个创建新 `HTMLElement` 的函数，不传参数时默认创建 `Div`；也可以选择传入子元素列表。此前你需要将其定义为：

```ts
declare function create(): Container<HTMLDivElement, HTMLDivElement[]>
declare function create<T extends HTMLElement>(element: T): Container<T, T[]>
declare function create<T extends HTMLElement, U extends HTMLElement>(
  element: T,
  children: U[],
): Container<T, U[]>
```

有了泛型参数默认值，可以简化为：

```ts
declare function create<T extends HTMLElement = HTMLDivElement, U = T[]>(
  element?: T,
  children?: U,
): Container<T, U>
```

泛型参数默认值遵循以下规则：

- 具有默认值的类型参数视为可选参数。
- 必选类型参数不得位于可选类型参数之后。
- 类型参数的默认类型必须满足该类型参数的约束（若有）。
- 在指定类型实参时，只需为必选类型参数指定实参，未指定的类型参数将解析为其默认类型。
- 若指定了默认类型，且类型推断无法确定候选类型，则使用默认类型。
- 与现有类或接口声明合并的类或接口声明，可以为现有类型参数引入默认值。
- 与现有类或接口声明合并的类或接口声明，可以引入新的类型参数，前提是该参数指定了默认值。

## 新增 `--strict` 主选项

TypeScript 新增的检查项通常默认关闭，以避免破坏现有项目。尽管避免破坏性变更是好事，但这一策略带来的问题是：要达到最高级别的类型安全变得越来越复杂，且每次 TypeScript 发布都需要显式地逐一启用。通过 [`strict`](/tsconfig#strict) 选项，可以一键选择最高类型安全级别，同时也接受随着新版编译器改进类型检查而可能报告更多错误。

新增的 [`strict`](/tsconfig#strict) 编译器选项代表了一组类型检查选项的推荐配置。具体来说，指定 [`strict`](/tsconfig#strict) 等同于同时指定以下所有选项（未来可能包含更多）：

- [`strictNullChecks`](/tsconfig#strictNullChecks)
- [`noImplicitAny`](/tsconfig#noImplicitAny)
- [`noImplicitThis`](/tsconfig#noImplicitThis)
- [`alwaysStrict`](/tsconfig#alwaysStrict)

从严格意义上说，[`strict`](/tsconfig#strict) 选项会设置以上编译器选项的_默认值_，各选项仍可单独控制。例如，

```sh
--strict --noImplicitThis false
```

会启用所有严格选项，但_不包括_ [`noImplicitThis`](/tsconfig#noImplicitThis) 选项。通过这种方式，可以表达"启用所有严格选项，但明确排除某些选项"的配置——换句话说，现在可以默认使用最高类型安全级别，同时有选择地退出特定检查。

从 TypeScript 2.3 起，`tsc --init` 生成的默认 `tsconfig.json` 在 `"compilerOptions"` 部分包含了 `"strict": true` 配置。因此，通过 `tsc --init` 创建的新项目默认会启用最高级别的类型安全。

## 增强的 `--init` 输出

除了默认启用 [`strict`](/tsconfig#strict) 外，`tsc --init` 的输出也得到了增强。`tsc --init` 生成的默认 `tsconfig.json` 文件现在包含一组常用编译器选项及其注释掉的描述说明。只需取消注释所需配置即可启用对应功能；新的输出旨在简化新项目的搭建流程，并在项目成长过程中保持配置文件的可读性。

## 使用 `--checkJs` 检查 .js 文件中的错误

默认情况下，TypeScript 编译器不会报告 .js 文件中的任何错误，即便使用了 [`allowJs`](/tsconfig#allowJs)。TypeScript 2.3 引入了 [`checkJs`](/tsconfig#checkJs) 选项，可以对 `.js` 文件进行类型检查并报告错误。

可以通过在文件中添加 `// @ts-nocheck` 注释来跳过特定文件的检查；反之，也可以不设置 [`checkJs`](/tsconfig#checkJs)，而仅在需要检查的 `.js` 文件中添加 `// @ts-check` 注释来启用检查。还可以在某行前加上 `// @ts-ignore` 来忽略该行的错误。

`.js` 文件的检查仍仅限于标准 ECMAScript 特性，类型注解只能用于 `.ts` 文件，在 `.js` 文件中会被标记为错误。可以使用 JSDoc 注释为 JavaScript 代码添加类型信息，详情请参阅 [JSDoc 支持文档](https://github.com/Microsoft/TypeScript/wiki/JSDoc-support-in-JavaScript)。

更多详情请参阅 [JavaScript 文件类型检查文档](https://github.com/Microsoft/TypeScript/wiki/Type-Checking-JavaScript-Files)。
