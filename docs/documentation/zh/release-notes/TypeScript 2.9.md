---
title: TypeScript 2.9
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-2-9.html
oneline: TypeScript 2.9 发布说明
---

## `keyof` 和映射类型支持 `number` 与 `symbol` 命名属性

TypeScript 2.9 在索引类型和映射类型中新增了对 `number` 和 `symbol` 命名属性的支持。
此前，`keyof` 运算符和映射类型仅支持 `string` 命名属性。

主要变更如下：

- 对于某个类型 `T`，其索引类型 `keyof T` 现在是 `string | number | symbol` 的子类型。
- 映射类型 `{ [P in K]: XXX }` 允许 `K` 为任何可赋值给 `string | number | symbol` 的类型。
- 在对泛型类型 `T` 的对象使用 `for...in` 语句时，迭代变量的推断类型此前为 `keyof T`，现在变更为 `Extract<keyof T, string>`（即 `keyof T` 中仅包含字符串类型的子集）。

对于对象类型 `X`，`keyof X` 的解析规则如下：

- 若 `X` 包含字符串索引签名，则 `keyof X` 为 `string`、`number` 以及所有 symbol-like 属性字面量类型的联合类型；
- 若 `X` 包含数字索引签名，则 `keyof X` 为 `number` 以及所有 string-like 和 symbol-like 属性字面量类型的联合类型；
- 否则，`keyof X` 为所有 string-like、number-like 和 symbol-like 属性字面量类型的联合类型。

其中：

- 对象类型的 string-like 属性，是指通过标识符、字符串字面量或字符串字面量类型的计算属性名声明的属性。
- 对象类型的 number-like 属性，是指通过数字字面量或数字字面量类型的计算属性名声明的属性。
- 对象类型的 symbol-like 属性，是指通过唯一 symbol 类型的计算属性名声明的属性。

在映射类型 `{ [P in K]: XXX }` 中，`K` 里的每个字符串字面量类型会引入一个字符串名称属性，每个数字字面量类型会引入一个数字名称属性，每个唯一 symbol 类型会引入一个唯一 symbol 名称属性。
此外，若 `K` 包含 `string` 类型，则会引入字符串索引签名；若 `K` 包含 `number` 类型，则会引入数字索引签名。

##### 示例

```ts
const c = 'c'
const d = 10
const e = Symbol()

const enum E1 {
  A,
  B,
  C,
}
const enum E2 {
  A = 'A',
  B = 'B',
  C = 'C',
}

type Foo = {
  a: string // 类似字符串的名称
  5: string // 类似数字的名称
  [c]: string // 类似字符串的名称
  [d]: string // 类似数字的名称
  [e]: string // 类似 Symbol 的名称
  [E1.A]: string // 类似数字的名称
  [E2.A]: string // 类似字符串的名称
}

type K1 = keyof Foo // "a" | 5 | "c" | 10 | typeof e | E1.A | E2.A
type K2 = Extract<keyof Foo, string> // "a" | "c" | E2.A
type K3 = Extract<keyof Foo, number> // 5 | 10 | E1.A
type K4 = Extract<keyof Foo, symbol> // typeof e
```

由于 `keyof` 现在通过在键类型中包含 `number` 来反映数字索引签名的存在，`Partial<T>` 和 `Readonly<T>` 等映射类型在应用于带有数字索引签名的对象类型时可以正确工作：

```ts
type Arrayish<T> = {
  length: number
  [x: number]: T
}

type ReadonlyArrayish<T> = Readonly<Arrayish<T>>

declare const map: ReadonlyArrayish<string>
let n = map.length
let x = map[123] // 此前类型为 any（或在启用 --noImplicitAny 时报错）
```

此外，随着 `keyof` 运算符支持 `number` 和 `symbol` 命名键，现在可以对通过数字字面量（如数字枚举类型）和唯一 symbol 索引的对象属性访问进行抽象。

```ts
const enum Enum {
  A,
  B,
  C,
}

const enumToStringMap = {
  [Enum.A]: 'Name A',
  [Enum.B]: 'Name B',
  [Enum.C]: 'Name C',
}

const sym1 = Symbol()
const sym2 = Symbol()
const sym3 = Symbol()

const symbolToNumberMap = {
  [sym1]: 1,
  [sym2]: 2,
  [sym3]: 3,
}

type KE = keyof typeof enumToStringMap // Enum（即 Enum.A | Enum.B | Enum.C）
type KS = keyof typeof symbolToNumberMap // typeof sym1 | typeof sym2 | typeof sym3

function getValue<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}

let x1 = getValue(enumToStringMap, Enum.C) // 返回 "Name C"
let x2 = getValue(symbolToNumberMap, sym3) // 返回 3
```

这是一项破坏性变更：此前，`keyof` 运算符和映射类型仅支持 `string` 命名属性。
原先假设 `keyof T` 的值始终为 `string` 的代码，现在将被标记为错误。

##### 示例

```ts
function useKey<T, K extends keyof T>(o: T, k: K) {
  var name: string = k // 错误：keyof T 不可赋值给 string
}
```

#### 建议

- 如果函数只能处理字符串命名的属性键，请在声明中使用 `Extract<keyof T, string>`：

  ```ts
  function useKey<T, K extends Extract<keyof T, string>>(o: T, k: K) {
    var name: string = k // 正常
  }
  ```

- 如果函数需要处理所有类型的属性键，则应在下游进行相应修改：

  ```ts
  function useKey<T, K extends keyof T>(o: T, k: K) {
    var name: string | number | symbol = k
  }
  ```

- 或者使用 [`keyofStringsOnly`](/tsconfig#keyofStringsOnly) 编译选项来禁用新行为。

## JSX 元素中的泛型类型参数

JSX 元素现在支持向泛型组件传递类型参数。

##### 示例

```ts
class GenericComponent<P> extends React.Component<P> {
  internalProp: P;
}

type Props = { a: number; b: string };

const x = <GenericComponent<Props> a={10} b="hi" />; // 正常

const y = <GenericComponent<Props> a={10} b={20} />; // 错误
```

## 泛型标签模板中的泛型类型参数

标签模板是 ECMAScript 2015 引入的一种调用形式。
与普通的函数调用类似，泛型函数可以在标签模板中使用，TypeScript 会推断其类型参数。

TypeScript 2.9 允许向标签模板字符串传递泛型类型参数。

##### 示例

```ts
declare function styledComponent<Props>(
  strs: TemplateStringsArray,
): Component<Props>

interface MyProps {
  name: string
  age: number
}

styledComponent<MyProps>`
  font-size: 1.5em;
  text-align: center;
  color: palevioletred;
`

declare function tag<T>(strs: TemplateStringsArray, ...args: T[]): T

// 推断失败，因为 'number' 和 'string' 都是相互冲突的候选类型
let a = tag<string | number>`${100} ${'hello'}`
```

## `import` 类型

模块可以导入其他模块中声明的类型，但非模块的全局脚本无法访问模块中声明的类型。`import` 类型正是为此而生。

在类型注解中使用 `import("mod")`，可以在不导入模块的情况下访问该模块导出的声明。

##### 示例

给定模块文件中 `Pet` 类的声明：

```ts
// module.d.ts

export declare class Pet {
  name: string
}
```

可以在非模块文件 `global-script.ts` 中使用：

```ts
// global-script.ts

function adopt(p: import('./module').Pet) {
  console.log(`Adopting ${p.name}...`)
}
```

在 JSDoc 注释中同样适用，可在 `.js` 文件中引用其他模块的类型：

```js
// a.js

/**
 * @param p { import("./module").Pet }
 */
function walk(p) {
  console.log(`Walking ${p.name}...`)
}
```

## 放宽声明文件生成的可见性规则

有了 `import` 类型后，编译器可以在不修改源代码的情况下，自动处理声明文件生成期间报告的许多可见性错误。

例如：

```ts
import { createHash } from 'crypto'

export const hash = createHash('sha256')
//           ^^^^
// 导出的变量 'hash' 拥有或正在使用来自外部模块 "crypto" 的名称 'Hash'，但无法被命名。
```

在 TypeScript 2.9 中，上述错误不再出现，生成的文件如下：

```ts
export declare const hash: import('crypto').Hash
```

## 支持 `import.meta`

TypeScript 2.9 引入了对 `import.meta` 的支持。这是一个新的元属性，由当前的 [TC39 提案](https://github.com/tc39/proposal-import-meta) 定义。

`import.meta` 的类型是全局的 `ImportMeta` 类型，定义于 `lib.es5.d.ts` 中。
该接口目前功能极为有限。
如需为 Node 或浏览器添加常用属性，需要通过接口合并，并视情况进行全局扩充。

##### 示例

假设 `import.meta` 上始终存在 `__dirname`，可通过重新打开 `ImportMeta` 接口来声明：

```ts
// node.d.ts
interface ImportMeta {
  __dirname: string
}
```

使用方式如下：

```ts
import.meta.__dirname // 类型为 'string'
```

`import.meta` 仅在目标为 `ESNext` 模块和 ECMAScript 目标时才被允许使用。

## 新增 `--resolveJsonModule`

在 Node.js 应用中，经常需要使用 `.json` 文件。TypeScript 2.9 新增了 [`resolveJsonModule`](/tsconfig#resolveJsonModule) 选项，支持导入 `.json` 文件、从中提取类型并生成相应的声明文件。

##### 示例

```ts
// settings.json

{
    "repo": "TypeScript",
    "dry": false,
    "debug": false
}
```

```ts
// a.ts

import settings from './settings.json'

settings.debug === true // 正常
settings.dry === 2 // 错误：运算符 '===' 不能应用于 boolean 和 number
```

```json tsconfig
// tsconfig.json

{
  "compilerOptions": {
    "module": "commonjs",
    "resolveJsonModule": true,
    "esModuleInterop": true
  }
}
```

## `--pretty` 输出默认启用

从 TypeScript 2.9 开始，若输出设备支持彩色文本，错误信息将默认以 [`pretty`](/tsconfig#pretty) 模式显示。
TypeScript 会检测输出流是否设置了 [`isTty`](https://nodejs.org/api/tty.html) 属性。

在命令行中使用 `--pretty false`，或在 `tsconfig.json` 中设置 `"pretty": false`，可以关闭 [`pretty`](/tsconfig#pretty) 输出。

## 新增 `--declarationMap`

同时启用 [`declarationMap`](/tsconfig#declarationMap) 和 [`declaration`](/tsconfig#declaration) 后，编译器会在输出 `.d.ts` 文件的同时生成对应的 `.d.ts.map` 文件。
语言服务现在也能识别这些 map 文件，并在可用时将基于声明文件的定义位置映射回原始源码。

换言之，对通过 [`declarationMap`](/tsconfig#declarationMap) 生成的 `.d.ts` 文件中的声明执行"转到定义"操作时，将跳转到该声明所在的源文件（`.ts`）位置，而非 `.d.ts` 文件。
