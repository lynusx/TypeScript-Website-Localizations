---
title: TypeScript 3.5
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-3-5.html
oneline: TypeScript 3.5 发布说明
---

## 速度提升

TypeScript 3.5 引入了多项围绕类型检查与增量构建的优化。

### 类型检查提速

相比 TypeScript 3.4，TypeScript 3.5 包含多项用于提高类型检查效率的优化。
这些改进在编辑器场景中尤为显著，因为在编辑器中类型检查驱动着代码补全列表等关键操作。

### `--incremental` 改进

TypeScript 3.5 改进了 3.4 的 [`incremental`](/tsconfig#incremental) 构建模式，通过保存全局状态的计算信息——包括编译器设置、为何查找这些文件、文件在何处找到等。
在涉及数百个项目并以 `--build` 模式使用 TypeScript 项目引用的场景中，[我们发现与 TypeScript 3.4 相比，重新构建的时间最多可减少 68%](https://github.com/Microsoft/TypeScript/pull/31101)！

更多详情可参阅以下 Pull Request：

- [缓存模块解析结果（cache module resolution）](https://github.com/Microsoft/TypeScript/pull/31100)
- [缓存根据 `tsconfig.json` 计算的配置（cache settings calculated from `tsconfig.json`）](https://github.com/Microsoft/TypeScript/pull/31101)

## `Omit` 辅助类型

TypeScript 3.5 引入了新的 `Omit` 辅助类型，用于从原始类型中剔除某些属性并创建新类型。

```ts
type Person = {
  name: string
  age: number
  location: string
}

type QuantumPerson = Omit<Person, 'location'>

// 等价于
type QuantumPerson = {
  name: string
  age: number
}
```

在这里，我们通过 `Omit` 辅助类型复制了 `Person` 除 `location` 之外的所有属性。

更多详情，[请参阅 GitHub 上新增 `Omit` 的 Pull Request](https://github.com/Microsoft/TypeScript/pull/30552)，以及[在对象 rest 中使用 `Omit` 的改动](https://github.com/microsoft/TypeScript/pull/31134)。

### 改进联合类型中的超额属性检查

在 TypeScript 3.4 及更早版本中，某些超额属性在理应被禁止的情况下却被允许传入。
例如，即便对象字面量中的 `name` 属性类型与 `Point` 和 `Label` 均不匹配，TypeScript 3.4 依然允许了该错误的 `name` 属性。

```ts
type Point = {
  x: number
  y: number
}

type Label = {
  name: string
}

const thing: Point | Label = {
  x: 0,
  y: 0,
  name: true, // 糟糕！
}
```

此前，非可辨识联合类型（non-discriminated union）不会对其成员执行_任何_超额属性检查，导致类型错误的 `name` 属性蒙混过关。

在 TypeScript 3.5 中，类型检查器至少会验证所有提供的属性属于_某个_联合成员并且具有相应的类型，这意味着上面的示例现在会正确报错。

请注意，只要属性类型有效，仍然允许部分重叠。

```ts
const pl: Point | Label = {
  x: 0,
  y: 0,
  name: 'origin', // 正常
}
```

## `--allowUmdGlobalAccess` 标志

在 TypeScript 3.5 中，通过全新的 [`allowUmdGlobalAccess`](/tsconfig#allowUmdGlobalAccess) 标志，你现在可以在任何地方（甚至在模块中）引用如下 UMD 全局声明：

```
export as namespace foo;
```

该模式为混用第三方库带来了更大的灵活性，使得库所声明的全局变量始终可以被使用，即使在模块内部也是如此。

更多详情，[请参阅 GitHub 上的 Pull Request](https://github.com/Microsoft/TypeScript/pull/30776/files)。

## 更智能的联合类型检查

在 TypeScript 3.4 及更早版本中，以下示例会报错：

```ts
type S = { done: boolean; value: number }
type T = { done: false; value: number } | { done: true; value: number }

declare let source: S
declare let target: T

target = source
```

这是因为 `S` 既不能赋值给 `{ done: false, value: number }`，也不能赋值给 `{ done: true, value: number }`。
为什么呢？
因为 `S` 中的 `done` 属性不够具体——它是 `boolean`，而 `T` 的每个构成成员中的 `done` 属性具体为 `true` 或 `false`。
这就是我们所说的“孤立检查每个成员类型”：TypeScript 并不会简单地把每个属性联合起来看 `S` 是否可赋值给它。
如果这样做，一些错误的代码就会蒙混过关，如下所示：

```ts
interface Foo {
  kind: 'foo'
  value: string
}

interface Bar {
  kind: 'bar'
  value: number
}

function doSomething(x: Foo | Bar) {
  if (x.kind === 'foo') {
    x.value.toLowerCase()
  }
}

// 糟糕——幸好 TypeScript 在这里报错了！
doSomething({
  kind: 'foo',
  value: 123,
})
```

然而，对于最初的示例而言，这种检查显得有些过于严苛了。
如果推导出 `S` 的任意可能值的精确类型，你实际上会发现它与 `T` 中的类型完全匹配。

在 TypeScript 3.5 中，当赋值给包含像 `T` 中那样的判别属性（discriminant properties）的类型时，语言实际上_会_更进一步，将像 `S` 这样的类型分解为所有可能取值类型的联合。
在这种情况下，由于 `boolean` 是 `true` 和 `false` 的联合，`S` 将被视为 `{ done: false, value: number }` 和 `{ done: true, value: number }` 的联合。

更多详情，[请参阅 GitHub 上的原始 Pull Request](https://github.com/microsoft/TypeScript/pull/30779)。

## 泛型构造函数的更高阶类型推断

在 TypeScript 3.4 中，我们改进了泛型函数返回函数时的类型推断，例如：

```ts
function compose<T, U, V>(f: (x: T) => U, g: (y: U) => V): (x: T) => V {
  return (x) => g(f(x))
}
```

当其接收其他泛型函数作为参数时，例如：

```ts
function arrayify<T>(x: T): T[] {
  return [x]
}

type Box<U> = { value: U }
function boxify<U>(y: U): Box<U> {
  return { value: y }
}

let newFn = compose(arrayify, boxify)
```

旧版本语言会推断出一个相对无用的类型 `(x: {}) => Box<{}[]>`, 而 TypeScript 3.4 的推断使得 `newFn` 能够保持泛型。
其新类型为 `<T>(x: T) => Box<T[]>`.

TypeScript 3.5 将此行为推广到构造函数上。

```ts
class Box<T> {
  kind: 'box'
  value: T
  constructor(value: T) {
    this.value = value
  }
}

class Bag<U> {
  kind: 'bag'
  value: U
  constructor(value: U) {
    this.value = value
  }
}

function composeCtor<T, U, V>(
  F: new (x: T) => U,
  G: new (y: U) => V,
): (x: T) => V {
  return (x) => new G(new F(x))
}

let f = composeCtor(Box, Bag) // 类型为 '<T>(x: T) => Bag<Box<T>>'
let a = f(1024) // 类型为 'Bag<Box<number>>'
```

除了上述组合模式外，对泛型构造函数的新推断还意味着在 React 等 UI 库中操作类组件的函数能够更准确地操作泛型类组件。

```ts
type ComponentClass<P> = new (props: P) => Component<P>
declare class Component<P> {
  props: P
  constructor(props: P)
}

declare function myHoc<P>(C: ComponentClass<P>): ComponentClass<P>

type NestedProps<T> = { foo: number; stuff: T }

declare class GenericComponent<T> extends Component<NestedProps<T>> {}

// 类型为 'new <T>(props: NestedProps<T>) => Component<NestedProps<T>>'
const GenericComponent2 = myHoc(GenericComponent)
```

了解更多信息，[请参阅 GitHub 上的原始 Pull Request](https://github.com/microsoft/TypeScript/pull/31116)。
