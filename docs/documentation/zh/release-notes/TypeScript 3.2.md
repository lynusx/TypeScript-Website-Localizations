---
title: TypeScript 3.2
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-3-2.html
oneline: TypeScript 3.2 发布说明
---

## `strictBindCallApply`

TypeScript 3.2 引入了一个全新的编译器选项 [`strictBindCallApply`](/tsconfig#strictBindCallApply)（属于 [`strict`](/tsconfig#strict) 选项家族），开启后函数对象上的 `bind`、`call` 和 `apply` 方法将获得强类型支持并进行严格检查。

```ts
function foo(a: number, b: string): string {
  return a + b
}

let a = foo.apply(undefined, [10]) // error: too few arguments
let b = foo.apply(undefined, [10, 20]) // error: 2nd argument is a number
let c = foo.apply(undefined, [10, 'hello', 30]) // error: too many arguments
let d = foo.apply(undefined, [10, 'hello']) // okay! returns a string
```

这是通过在 `lib.d.ts` 中引入两个新类型 `CallableFunction` 和 `NewableFunction` 来实现的。这两个类型分别针对常规函数和构造函数，包含了专用于 `bind`、`call` 和 `apply` 的泛型方法声明。这些声明利用泛型 rest 参数（参见 #24897）以强类型方式捕获并反映参数列表。在 [`strictBindCallApply`](/tsconfig#strictBindCallApply) 模式下，这些声明将取代 `Function` 类型所提供的（非常宽泛的）声明。

## 注意事项

由于更严格的检查可能会暴露此前未报告的错误，因此在 [`strict`](/tsconfig#strict) 模式下这是一项破坏性变更。

此外，该新功能的[另一项注意事项](https://github.com/Microsoft/TypeScript/pull/27028#issuecomment-429334450)是，受某些限制影响，`bind`、`call` 和 `apply` 尚无法对泛型函数或包含重载的函数进行完全建模。
当在泛型函数上使用这些方法时，类型参数会被替换为空对象类型（`{}`）；而在具有重载的函数上使用时，将仅对最后一个重载进行建模。

## 对象字面量中的泛型展开表达式

在 TypeScript 3.2 中，对象字面量现在允许包含泛型展开表达式，并会生成交叉类型，这与 `Object.assign` 函数和 JSX 字面量类似。例如：

```ts
function taggedObject<T, U extends string>(obj: T, tag: U) {
  return { ...obj, tag } // T & { tag: U }
}

let x = taggedObject({ x: 10, y: 20 }, 'point') // { x: number, y: number } & { tag: "point" }
```

在泛型展开表达式两侧的属性赋值和非泛型展开表达式会尽可能地进行合并。例如：

```ts
function foo1<T>(t: T, obj1: { a: string }, obj2: { b: string }) {
  return { ...obj1, x: 1, ...t, ...obj2, y: 2 } // { a: string, x: number } & T & { b: string, y: number }
}
```

非泛型展开表达式继续按原方式处理：调用和构造签名会被剥离，仅保留非方法属性，并且对于同名属性，将采用最右侧属性的类型。这与交叉类型形成了对比，交叉类型会串联调用和构造签名，保留所有属性，并将同名属性的类型进行交叉。因此，相同的类型在通过泛型类型实例化创建时，展开可能会产生不同的结果：

```ts
function spread<T, U>(t: T, u: U) {
  return { ...t, ...u } // T & U
}

declare let x: { a: string; b: number }
declare let y: { b: string; c: boolean }

let s1 = { ...x, ...y } // { a: string, b: string, c: boolean }
let s2 = spread(x, y) // { a: string, b: number } & { b: string, c: boolean }
let b1 = s1.b // string
let b2 = s2.b // number & string
```

## 泛型对象剩余变量与参数

TypeScript 3.2 还允许从泛型变量中解构出 rest 绑定。这是通过使用 `lib.d.ts` 中预定义的 `Pick` 和 `Exclude` 辅助类型，并结合目标泛型类型以及解构模式中其他绑定的名称来实现的。

```ts
function excludeTag<T extends { tag: string }>(obj: T) {
  let { tag, ...rest } = obj
  return rest // Pick<T, Exclude<keyof T, "tag">>
}

const taggedPoint = { x: 10, y: 20, tag: 'point' }
const point = excludeTag(taggedPoint) // { x: number, y: number }
```

## BigInt

BigInt 是 ECMAScript 中一项即将到来的提案，使我们能够在理论上对任意大的整数进行建模。
TypeScript 3.2 带来了对 BigInt 的类型检查，并在目标为 `esnext` 时支持生成 BigInt 字面量。

TypeScript 中的 BigInt 支持引入了一个名为 `bigint`（全部小写）的新原始类型。
你可以通过调用 `BigInt()` 函数，或者在任何整数字面量末尾添加 `n` 写出 BigInt 字面量来获取 `bigint`：

```ts
let foo: bigint = BigInt(100) // the BigInt function
let bar: bigint = 100n // a BigInt literal

// *Slaps roof of fibonacci function*
// This bad boy returns ints that can get *so* big!
function fibonacci(n: bigint) {
  let result = 1n
  for (let last = 0n, i = 0n; i < n; i++) {
    const current = result
    result += last
    last = current
  }
  return result
}

fibonacci(10000n)
```

你可能会认为 `number` 和 `bigint` 之间存在紧密的交互，但两者实际上属于完全独立的领域。

```ts
declare let foo: number
declare let bar: bigint

foo = bar // error: Type 'bigint' is not assignable to type 'number'.
bar = foo // error: Type 'number' is not assignable to type 'bigint'.
```

正如 ECMAScript 规范所规定，在算术运算中混合使用 `number` 和 `bigint` 是错误的。
必须显式将值转换为 `BigInt`。

```ts
console.log(3.141592 * 10000n) // error
console.log(3145 * 10n) // error
console.log(BigInt(3145) * 10n) // okay!
```

同样需要注意的是，使用 `typeof` 运算符时，`bigint` 会产生一个新的字符串：字符串 `"bigint"`。
因此，TypeScript 如你所期望的那样能够通过 `typeof` 正确进行类型收窄。

```ts
function whatKindOfNumberIsIt(x: number | bigint) {
  if (typeof x === 'bigint') {
    console.log("'x' is a bigint!")
  } else {
    console.log("'x' is a floating-point number")
  }
}
```

我们衷心感谢 [Caleb Sander](https://github.com/calebsander) 在该特性上所付出的所有努力。
我们对这项贡献深表感激，相信广大用户也是如此！

## 注意事项

正如我们所提到的，BigInt 支持仅在目标为 `esnext` 时可用。
可能不那么明显的是，由于 BigInt 在针对 `+`、`-`、`*` 等数学运算符时具有不同的行为，为尚不存在该特性的旧目标（例如 `es2017` 及以下）提供支持将涉及重写每一个运算。
TypeScript 需要根据类型分发到正确的行为，因此每一次加法、字符串拼接、乘法等操作都将涉及函数调用。

出于这个原因，我们目前没有提供降级编译（downleveling）支持的计划。
好的一面是，Node 11 和较新版本的 Chrome 已经支持此特性，因此在以 `esnext` 为目标时，你可以在这些环境中使用 BigInt。

某些运行目标可能包含 polyfill 或类似 BigInt 的运行时对象。
针对这些场景，你可能需要在编译器选项中的 [`lib`](/tsconfig#lib) 设置添加 `esnext.bigint`。

## 非单元类型作为联合判别属性

TypeScript 3.2 通过放宽对判别属性（discriminant property）的判定规则，让类型收窄变得更加容易。
联合类型的公共属性只要包含_某种_单例类型（例如字符串字面量、`null` 或 `undefined`），且不包含泛型，现在就会被视为判别属性。

因此，在以下示例中，TypeScript 3.2 会将 `error` 属性视为判别属性，而此前则不会，因为 `Error` 不是单例类型。
得益于此，在 `unwrap` 函数体内类型收窄得以正常工作。

```ts
type Result<T> = { error: Error; data: null } | { error: null; data: T }

function unwrap<T>(result: Result<T>) {
  if (result.error) {
    // Here 'error' is non-null
    throw result.error
  }

  // Now 'data' is non-null
  return result.data
}
```

## 通过 Node.js 包继承 `tsconfig.json`

TypeScript 3.2 现在支持从 `node_modules` 解析 `tsconfig.json`。当在 `tsconfig.json` 的 `extends` 字段中使用裸路径（bare path）时，TypeScript 将会深入 `node_modules` 包进行查找。

```jsonc tsconfig
{
  "extends": "@my-team/tsconfig-base",
  "include": ["./**/*"],
  "compilerOptions": {
    // Override certain options on a project-by-project basis.
    "strictBindCallApply": false,
  },
}
```

在此处，TypeScript 会向上遍历 `node_modules` 文件夹以查找 `@my-team/tsconfig-base` 包。对于找到的每个包，TypeScript 会首先检查 `package.json` 是否包含 `"tsconfig"` 字段；如果包含，TypeScript 将尝试从该字段指定的文件加载配置。如果两者均不存在，TypeScript 将尝试从包根目录下的 `tsconfig.json` 读取。这与 Node 解析包中 `.js` 文件的查找过程以及 TypeScript 现有的 `.d.ts` 查找过程非常相似。

这一特性对于大型组织或拥有大量分布式依赖项的项目极其有用。

## 新增 `--showConfig` 标志

TypeScript 编译器 `tsc` 支持了一个名为 `--showConfig` 的新标志。
运行 `tsc --showConfig` 时，TypeScript 会计算出最终生效的 `tsconfig.json`（在计算了从 `extends` 字段继承的选项之后）并将其打印输出。
这在诊断通用的配置问题时非常有用。

## JavaScript 中的 `Object.defineProperty` 声明

在编写 JavaScript 文件时（使用 [`allowJs`](/tsconfig#allowJs)），TypeScript 现在能够识别使用 `Object.defineProperty` 的声明。
这意味着在 JavaScript 文件中启用类型检查时（通过开启 [`checkJs`](/tsconfig#checkJs) 选项或在文件顶部添加 `// @ts-check` 注释），你将获得更佳的代码补全和更强的类型检查支持。

```js
// @ts-check

let obj = {}
Object.defineProperty(obj, 'x', { value: 'hello', writable: false })

obj.x.toLowercase()
//    ~~~~~~~~~~~
//    error:
//     Property 'toLowercase' does not exist on type 'string'.
//     Did you mean 'toLowerCase'?

obj.x = 'world'
//  ~
//  error:
//   Cannot assign to 'x' because it is a read-only property.
```
