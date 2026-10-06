---
title: TypeScript 2.4
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-2-4.html
oneline: TypeScript 2.4 发布说明
---

## 动态导入表达式

动态 `import` 表达式是 ECMAScript 的一项新特性，允许用户在程序的任意位置异步地按需加载模块。

这意味着你可以有条件地、延迟地导入其他模块和库。例如，以下 `async` 函数仅在需要时才导入工具库：

```ts
async function getZipFile(name: string, files: File[]): Promise<File> {
  const zipUtil = await import('./utils/create-zip-file')
  const zipContents = await zipUtil.getContentAsBlob(files)
  return new File(zipContents, name)
}
```

许多打包工具支持基于 `import` 表达式自动拆分输出包，因此建议在使用 `esnext` 模块目标时搭配此特性。

## 字符串枚举

TypeScript 2.4 现在允许枚举成员使用字符串初始值。

```ts
enum Colors {
  Red = 'RED',
  Green = 'GREEN',
  Blue = 'BLUE',
}
```

需要注意的是，使用字符串初始值的枚举无法进行反向映射以获取原始枚举成员名称。换句话说，不能通过 `Colors["RED"]` 得到字符串 `"Red"`。

## 改进的泛型推断

TypeScript 2.4 在泛型推断方面引入了几项重要改进。

### 将返回类型作为推断目标

TypeScript 现在可以对调用的返回类型进行推断，这能改善开发体验并发现更多错误。以下写法现在可以正常工作：

```ts
function arrayMap<T, U>(f: (x: T) => U): (a: T[]) => U[] {
  return (a) => a.map(f)
}

const lengths: (a: string[]) => number[] = arrayMap((s) => s.length)
```

以下是一个因此能被发现的新错误示例：

```ts
let x: Promise<string> = new Promise((resolve) => {
  resolve(10)
  //      ~~ 错误！
})
```

### 从上下文类型推断类型参数

在 TypeScript 2.4 之前，对于以下示例：

```ts
let f: <T>(x: T) => T = (y) => y
```

`y` 的类型会被推断为 `any`。这意味着程序可以通过类型检查，但实际上可以对 `y` 做任何操作，例如：

```ts
let f: <T>(x: T) => T = (y) => y() + y.foo.bar
```

最后这个例子实际上并不类型安全。

在 TypeScript 2.4 中，右侧函数会隐式地_获取_类型参数，`y` 会被推断为对应类型参数的类型。若以类型参数约束不支持的方式使用 `y`，则会正确地报错。在这个例子中，`T` 的约束（隐式地）为 `{}`，因此最后那个例子会如预期地报错。

### 更严格的泛型函数检查

TypeScript 现在在比较两个单签名类型时会尝试统一类型参数。因此，在关联两个泛型签名时，你将获得更严格的检查，有助于发现潜在的错误。

```ts
type A = <T, U>(x: T, y: U) => [T, U]
type B = <S>(x: S, y: S) => [S, S]

function f(a: A, b: B) {
  a = b // 错误
  b = a // 正常
}
```

## 回调参数的严格逆变检查

TypeScript 一直以双变（bivariant）方式比较参数类型。这样做有多方面原因，但在涉及 `Promise` 和 `Observable` 时，其负面影响逐渐显现。

TypeScript 2.4 在关联两个回调类型时收紧了这一行为。例如：

```ts
interface Mappable<T> {
  map<U>(f: (x: T) => U): Mappable<U>
}

declare let a: Mappable<number>
declare let b: Mappable<string | number>

a = b
b = a
```

在 TypeScript 2.4 之前，这段代码可以通过。在关联 `map` 的类型时，TypeScript 会以双变方式关联其参数（即 `f` 的类型），进而以双变方式关联这些参数的类型。

在 TypeScript 2.4 中，关联 `map` 的类型时，语言会检查每个参数是否为回调类型，若是，则确保这些参数在当前关系下以逆变方式进行检查。

换句话说，TypeScript 现在能发现上述错误。这可能是一个破坏性变更，但通常利大于弊。

## 弱类型检测

TypeScript 2.4 引入了"弱类型"的概念。若某个类型的所有属性均为可选属性，则认为它是_弱类型_。例如，下面的 `Options` 类型就是一个弱类型：

```ts
interface Options {
  data?: string
  timeout?: number
  maxRetries?: number
}
```

在 TypeScript 2.4 中，若赋值的对象与弱类型没有任何属性重叠，则该赋值现在会报错。例如：

```ts
function sendMessage(options: Options) {
  // ...
}

const opts = {
  payload: 'hello world!',
  retryOnFail: true,
}

// 错误！
sendMessage(opts)
// 'opts' 的类型与 'Options' 本身之间没有重叠。
// 也许我们本意是想使用 'data'/'maxRetries' 而不是 'payload'/'retryOnFail'。
```

可以将此理解为 TypeScript 对弱类型"加固"了保证，从而捕获那些原本会被忽略的静默错误。

由于这是一个破坏性变更，你可能需要了解相应的解决方案，与严格对象字面量检查的解决方案相同：

1. 若属性确实存在，则声明它们。
2. 为弱类型添加索引签名（即 `[propName: string]: {}`）。
3. 使用类型断言（即 `opts as Options`）。
