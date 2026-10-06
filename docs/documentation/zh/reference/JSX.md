---
title: JSX
layout: docs
permalink: /zh/docs/handbook/jsx.html
oneline: 在 TypeScript 中使用 JSX
translatable: true
---

[JSX](https://facebook.github.io/jsx/) 是一种可嵌入的类似 XML 的语法。
它旨在被转换为合法的 JavaScript，不过转换的具体语义取决于具体实现。
JSX 随着 [React](https://reactjs.org/) 框架的流行而普及，但此后也出现了其他实现。
TypeScript 支持直接将 JSX 嵌入代码、进行类型检查以及将其编译为 JavaScript。

## 基本用法

为了使用 JSX，你必须完成以下两件事：

1. 将文件命名为 `.tsx` 扩展名
2. 启用 [`jsx`](/tsconfig#jsx) 选项

TypeScript 自带多种 JSX 模式：`preserve`、`react`（经典运行时）、`react-jsx`（自动运行时）、`react-jsxdev`（自动开发运行时）以及 `react-native`。
`preserve` 模式会将 JSX 保留在输出中，以供后续其他转换步骤（例如 [Babel](https://babeljs.io/)）进一步处理。
此外，输出文件的扩展名为 `.jsx`。
`react` 模式会生成 `React.createElement`，无需在使用前进行 JSX 转换，输出文件的扩展名为 `.js`。
`react-native` 模式等同于 `preserve`，因为它会保留所有 JSX，但输出文件的扩展名为 `.js`。

| 模式           | 输入      | 输出                                              | 输出文件扩展名 |
| -------------- | --------- | ------------------------------------------------- | -------------- |
| `preserve`     | `<div />` | `<div />`                                         | `.jsx`         |
| `react`        | `<div />` | `React.createElement("div")`                      | `.js`          |
| `react-native` | `<div />` | `<div />`                                         | `.js`          |
| `react-jsx`    | `<div />` | `_jsx("div", {}, void 0);`                        | `.js`          |
| `react-jsxdev` | `<div />` | `_jsxDEV("div", {}, void 0, false, {...}, this);` | `.js`          |

你可以使用 [`jsx`](/tsconfig#jsx) 命令行标志或 [tsconfig.json 文件中的对应 `jsx` 选项](/tsconfig#jsx)来指定此模式。

> \*注意：当目标为 react JSX 生成时，你可以使用 [`jsxFactory`](/tsconfig#jsxFactory) 选项指定要使用的 JSX 工厂函数（默认为 `React.createElement`）。

## `as` 运算符

回顾一下如何编写类型断言：

```ts
const foo = <Foo>bar
```

这会将变量 `bar` 断言为具有类型 `Foo`。
由于 TypeScript 同样使用尖括号进行类型断言，将其与 JSX 语法结合会带来某些语法解析上的困难。因此，TypeScript 在 `.tsx` 文件中禁止使用尖括号类型断言。

由于上述语法不能在 `.tsx` 文件中使用，因此应使用另一种类型断言运算符：`as`。
上面的示例可以轻松使用 `as` 运算符重写：

```ts
const foo = bar as Foo
```

`as` 运算符在 `.ts` 和 `.tsx` 文件中均可使用，其行为与尖括号风格的类型断言完全相同。

## 类型检查

为了理解 JSX 的类型检查，你首先必须理解内置元素（intrinsic elements）与基于值的元素（value-based elements）之间的区别。
对于一个 JSX 表达式 `<expr />`，`expr` 既可以指代环境内置的内容（例如 DOM 环境中的 `div` 或 `span`），也可以指代你创建的自定义组件。
这一点之所以重要，主要有两个原因：

1. 对于 React 而言，内置元素会作为字符串输出（`React.createElement("div")`），而自定义组件则不会（`React.createElement(MyComponent)`）。
2. 在 JSX 元素中传递的属性类型查找方式应当不同。
   内置元素的属性应当是*内置已知*的，而组件通常希望指定它们自己的一组属性。

TypeScript 采用了[与 React 相同的约定](http://facebook.github.io/react/docs/jsx-in-depth.html#html-tags-vs.-react-components)来区分这两者：
内置元素始终以小写字母开头，而基于值的元素始终以大写字母开头。

### `JSX` 命名空间

TypeScript 中的 JSX 由 `JSX` 命名空间进行类型定义。根据 `jsx` 编译器选项的不同，`JSX` 命名空间可以在不同的位置进行定义。

`jsx` 选项 `preserve`、`react` 和 `react-native` 使用经典运行时的类型定义。这意味着作用域内需要存在一个由 `jsxFactory` 编译器选项决定的变量。`JSX` 命名空间应该指定在 JSX 工厂的最顶层标识符上。例如，React 使用默认工厂 `React.createElement`，这意味着其 `JSX` 命名空间应定义为 `React.JSX`。

```ts
export function createElement(): any

export namespace JSX {
  // …
}
```

使用者应该始终将 React 导入为 `React`：

```ts
import * as React from 'react'
```

Preact 使用 JSX 工厂 `h`。这意味着它的类型应该定义为 `h.JSX`。

```ts
export function h(props: any): any

export namespace h.JSX {
  // …
}
```

使用者应该使用具名导入来导入 `h`：

```ts
import { h } from 'preact'
```

对于 `jsx` 选项 `react-jsx` 和 `react-jsxdev`，`JSX` 命名空间应该从匹配的入口点导出。对于 `react-jsx`，这是 `${jsxImportSource}/jsx-runtime`；对于 `react-jsxdev`，则是 `${jsxImportSource}/jsx-dev-runtime`。由于这些路径不包含文件扩展名，因此你必须使用 `package.json` 中的 [`exports`](https://nodejs.org/api/packages.html#exports) 字段映射来支持 ESM 用户。

```json
{
  "exports": {
    "./jsx-runtime": "./jsx-runtime.js",
    "./jsx-dev-runtime": "./jsx-dev-runtime.js"
  }
}
```

然后在 `jsx-runtime.d.ts` 和 `jsx-dev-runtime.d.ts` 中：

```ts
export namespace JSX {
  // …
}
```

请注意，虽然导出 `JSX` 命名空间足以进行类型检查，但在生产运行时需要运行时的 `jsx`、`jsxs` 和 `Fragment` 导出，而在开发运行时需要 `jsxDEV` 和 `Fragment`。理想情况下，你也应该为它们添加类型。

如果 `JSX` 命名空间在对应位置不可用，经典运行时和自动运行时都会回退到全局的 `JSX` 命名空间。

### 内置元素

内置元素会在特殊接口 `JSX.IntrinsicElements` 上查找。
默认情况下，如果未指定该接口，则允许使用任何元素，且不会对内置元素进行类型检查。
然而，如果该接口*存在*，则内置元素的名称将作为属性在 `JSX.IntrinsicElements` 接口上进行查找。
例如：

```tsx
declare namespace JSX {
  interface IntrinsicElements {
    foo: any
  }
}

;<foo /> // 正常
;<bar /> // 错误
```

在上面的示例中，`<foo />` 可以正常工作，但 `<bar />` 会报错，因为它未在 `JSX.IntrinsicElements` 上指定。

> 注意：你还可以在 `JSX.IntrinsicElements` 上指定一个通用的字符串索引签名，如下所示：

```ts
declare namespace JSX {
  interface IntrinsicElements {
    [elemName: string]: any
  }
}
```

### 基于值的元素

基于值的元素仅根据作用域内的标识符进行查找。

```tsx
import MyComponent from './myComponent'

;<MyComponent /> // 正常
;<SomeOtherComponent /> // 错误
```

定义基于值的元素有两种方式：

1. 函数式组件（Function Component, FC）
2. 类组件（Class Component）

由于这两种基于值的元素在 JSX 表达式中彼此无法区分，因此 TS 首先尝试使用重载解析将表达式解析为函数式组件。如果该过程成功，则 TS 完成将表达式解析为其声明。如果未能将其解析为函数式组件，TS 随后将尝试将其解析为类组件。如果再次失败，TS 将报告错误。

#### 函数式组件

顾名思义，该组件定义为一个 JavaScript 函数，其第一个参数是一个 `props` 对象。
TS 强制要求其返回类型必须可赋值给 `JSX.Element`。

```tsx
interface FooProp {
  name: string
  X: number
  Y: number
}

declare function AnotherComponent(prop: { name: string })
function ComponentFoo(prop: FooProp) {
  return <AnotherComponent name={prop.name} />
}

const Button = (prop: { value: string }, context: { color: string }) => (
  <button />
)
```

由于函数式组件本质上就是一个 JavaScript 函数，因此也可以在这里使用函数重载：

```ts twoslash
// @noErrors
declare module JSX {
  interface Element {}
  interface IntrinsicElements {
    [s: string]: any
  }
}
// ---cut---
interface ClickableProps {
  children: JSX.Element[] | JSX.Element
}

interface HomeProps extends ClickableProps {
  home: JSX.Element
}

interface SideProps extends ClickableProps {
  side: JSX.Element | string
}

function MainButton(prop: HomeProps): JSX.Element
function MainButton(prop: SideProps): JSX.Element
function MainButton(prop: ClickableProps): JSX.Element {
  // ...
}
```

> 注意：函数式组件之前被称为无状态函数组件（Stateless Function Components, SFC）。由于在较新版本的 React 中函数式组件不再被视为无状态，类型 `SFC` 及其别名 `StatelessComponent` 已被废弃。

#### 类组件

我们可以定义类组件的类型。
然而，在定义之前，最好先理解两个新术语：_元素类类型_（element class type）与*元素实例类型*（element instance type）。

对于 `<Expr />`，*元素类类型*就是 `Expr` 的类型。
所以在上面的示例中，如果 `MyComponent` 是一个 ES6 类，则类类型就是该类的构造函数及其静态成员。
如果 `MyComponent` 是一个工厂函数，则类类型就是该函数本身。

类类型确定后，实例类型由该类类型的构造签名或调用签名（以存在者为准）的返回类型的联合决定。
同样，对于 ES6 类，实例类型就是该类实例的类型；对于工厂函数，则是该函数返回值的类型。

```ts
class MyComponent {
  render() {}
}

// 使用构造签名
const myComponent = new MyComponent()

// 元素类类型 => MyComponent
// 元素实例类型 => { render: () => void }

function MyFactoryFunction() {
  return {
    render: () => {},
  }
}

// 使用调用签名
const myComponent = MyFactoryFunction()

// 元素类类型 => MyFactoryFunction
// 元素实例类型 => { render: () => void }
```

元素实例类型值得注意的一点是，它必须能够赋值给 `JSX.ElementClass`，否则会导致错误。
默认情况下，`JSX.ElementClass` 为 `{}`，但可以通过对其进行扩展，将 JSX 的使用限制为仅符合相应接口的类型。

```tsx
declare namespace JSX {
  interface ElementClass {
    render: any
  }
}

class MyComponent {
  render() {}
}
function MyFactoryFunction() {
  return { render: () => {} }
}

;<MyComponent /> // 正常
;<MyFactoryFunction /> // 正常

class NotAValidComponent {}
function NotAValidFactoryFunction() {
  return {}
}

;<NotAValidComponent /> // 错误
;<NotAValidFactoryFunction /> // 错误
```

### 属性类型检查

对属性进行类型检查的第一步是确定*元素属性类型*（element attributes type）。
内置元素与基于值的元素在此过程上略有不同。

对于内置元素，它是 `JSX.IntrinsicElements` 上的属性类型：

```tsx
declare namespace JSX {
  interface IntrinsicElements {
    foo: { bar?: boolean }
  }
}

// 'foo' 的元素属性类型为 '{bar?: boolean}'
;<foo bar />
```

对于基于值的元素，情况稍显复杂。
它由先前确定的*元素实例类型*上的属性类型决定。
具体使用哪个属性由 `JSX.ElementAttributesProperty` 决定。
该接口应该只声明一个属性。
然后将使用该属性的名称。
自 TypeScript 2.8 起，如果未提供 `JSX.ElementAttributesProperty`，将改用类元素构造函数或函数式组件调用的第一个参数的类型。

```tsx
declare namespace JSX {
  interface ElementAttributesProperty {
    props // 指定要使用的属性名称
  }
}

class MyComponent {
  // 在元素实例类型上指定该属性
  props: {
    foo?: string
  }
}

// 'MyComponent' 的元素属性类型为 '{foo?: string}'
;<MyComponent foo="bar" />
```

元素属性类型用于对 JSX 中的属性进行类型检查。
支持可选属性和必填属性。

```tsx
declare namespace JSX {
  interface IntrinsicElements {
    foo: { requiredProp: string; optionalProp?: number }
  }
}

;<foo requiredProp="bar" /> // 正常
;<foo requiredProp="bar" optionalProp={0} /> // 正常
;<foo /> // 错误，缺少 requiredProp
;<foo requiredProp={0} /> // 错误，requiredProp 应为 string
;<foo requiredProp="bar" unknownProp /> // 错误，unknownProp 不存在
;<foo requiredProp="bar" some-unknown-prop /> // 正常，因为 'some-unknown-prop' 不是合法的标识符
```

> 注意：如果属性名不是合法的 JS 标识符（例如 `data-*` 属性），即便它未在元素属性类型中找到，也不会被视为错误。

此外，`JSX.IntrinsicAttributes` 接口可用于指定 JSX 框架所使用的额外属性，这些属性通常不被组件自身的 props 或参数所使用——例如 React 中的 `key`。更进一步，泛型 `JSX.IntrinsicClassAttributes<T>` 类型还可以专门用来为类组件（而非函数式组件）指定此类额外属性。在该类型中，泛型参数对应于类实例类型。在 React 中，这用于支持类型为 `Ref<T>` 的 `ref` 属性。通常来说，这些接口上的所有属性都应该是可选的，除非你希望使用你的 JSX 框架的用户在每个标签上都必须提供某个属性。

展开运算符（spread operator）也同样受支持：

```tsx
const props = { requiredProp: 'bar' }
;<foo {...props} /> // 正常

const badProps = {}
;<foo {...badProps} /> // 错误
```

### 子元素类型检查

在 TypeScript 2.3 中，TS 引入了对*子元素*（children）的类型检查。_children_ 是*元素属性类型*中的一个特殊属性，子 _JSXExpression_ 会被插入到该属性中。
类似于 TS 使用 `JSX.ElementAttributesProperty` 来确定 _props_ 的名称，TS 使用 `JSX.ElementChildrenAttribute` 来确定这些 props 中 _children_ 的名称。
`JSX.ElementChildrenAttribute` 应该只声明一个属性。

```ts
declare namespace JSX {
  interface ElementChildrenAttribute {
    children: {} // 指定要使用的 children 名称
  }
}
```

```tsx
<div>
  <h1>Hello</h1>
</div>;

<div>
  <h1>Hello</h1>
  World
</div>;

const CustomComp = (props) => <div>{props.children}</div>
<CustomComp>
  <div>Hello World</div>
  {"This is just a JS expression..." + 1000}
</CustomComp>
```

你可以像指定其他任何属性一样指定 _children_ 的类型。如果你使用了 [React 类型定义](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react)，这将会覆盖其中的默认类型。

```tsx
interface PropsType {
  children: JSX.Element
  name: string
}

class Component extends React.Component<PropsType, {}> {
  render() {
    return (
      <h2>
        {this.props.children}
      </h2>
    )
  }
}

// 正常
<Component name="foo">
  <h1>Hello World</h1>
</Component>

// 错误：children 的类型为 JSX.Element，而不是 JSX.Element 数组
<Component name="bar">
  <h1>Hello World</h1>
  <h2>Hello World</h2>
</Component>

// 错误：children 的类型为 JSX.Element，而不是 JSX.Element 数组或 string。
<Component name="baz">
  <h1>Hello</h1>
  World
</Component>
```

## JSX 结果类型

默认情况下，JSX 表达式的结果类型为 `any`。
你可以通过指定 `JSX.Element` 接口来自定义该类型。
然而，无法通过此接口检索有关 JSX 元素、属性或子元素的类型信息。
它是一个黑盒。

## JSX 函数返回类型

默认情况下，函数式组件必须返回 `JSX.Element | null`。然而，这并不总是能够代表运行时行为。自 TypeScript 5.1 起，你可以指定 `JSX.ElementType` 来覆盖合法的 JSX 组件类型。请注意，这并不能定义哪些 props 是合法的；props 的类型始终由传入组件的第一个参数决定。默认结构大致如下：

```ts
namespace JSX {
    export type ElementType =
        // 所有有效的小写标签
        | keyof IntrinsicElements
        // 函数式组件
        | (props: any) => Element
        // 类组件
        | new (props: any) => ElementClass;
    export interface IntrinsicAttributes extends /*...*/ {}
    export type Element = /*...*/;
    export type ElementClass = /*...*/;
}
```

## 嵌入表达式

JSX 允许你通过用花括号（`{ }`）包裹表达式，将表达式嵌入到标签之间。

```tsx
const a = (
  <div>
    {['foo', 'bar'].map((i) => (
      <span>{i / 2}</span>
    ))}
  </div>
)
```

上面的代码将会报错，因为你不能用字符串除以数字。
当使用 `preserve` 选项时，输出结果如下所示：

```tsx
const a = (
  <div>
    {['foo', 'bar'].map(function (i) {
      return <span>{i / 2}</span>
    })}
  </div>
)
```

## React 集成

要在 React 中使用 JSX，你应该使用 [React 类型定义](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react)。
这些类型定义为在 React 中使用适当地定义了 `JSX` 命名空间。

```tsx
/// <reference path="react.d.ts" />

interface Props {
  foo: string
}

class MyComponent extends React.Component<Props, {}> {
  render() {
    return <span>{this.props.foo}</span>
  }
}

;<MyComponent foo="bar" /> // 正常
;<MyComponent foo={0} /> // 错误
```

### 配置 JSX

有多个编译器标志可用于自定义你的 JSX，它们既可以作为编译器标志使用，也可以通过每个文件内联的编译指令（pragma）使用。要了解更多信息，请参阅它们的 tsconfig 参考页面：

- [`jsxFactory`](/tsconfig#jsxFactory)
- [`jsxFragmentFactory`](/tsconfig#jsxFragmentFactory)
- [`jsxImportSource`](/tsconfig#jsxImportSource)
