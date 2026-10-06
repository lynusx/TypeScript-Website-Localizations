---
title: TypeScript 5.0
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-5-0.html
oneline: TypeScript 5.0 发布说明
---

## 装饰器（Decorators）

装饰器（Decorators）是一项即将加入 ECMAScript 的特性，它允许我们以可复用的方式定制类及其成员。

让我们来看以下代码：

```ts
class Person {
  name: string
  constructor(name: string) {
    this.name = name
  }

  greet() {
    console.log(`Hello, my name is ${this.name}.`)
  }
}

const p = new Person('Ray')
p.greet()
```

这里的 `greet` 非常简单，但试想一下它要复杂得多——比如包含异步逻辑、递归调用、带有副作用等。
无论你面对的是怎样的一团乱麻，假设你添加了一些 `console.log` 调用来帮助调试 `greet`：

```ts
class Person {
  name: string
  constructor(name: string) {
    this.name = name
  }

  greet() {
    console.log('LOG: Entering method.')

    console.log(`Hello, my name is ${this.name}.`)

    console.log('LOG: Exiting method.')
  }
}
```

这种模式非常普遍。
如果能有一种方法对每个方法都统一做到这一点，那就太棒了！

这正是装饰器的用武之地。
我们可以编写一个名为 `loggedMethod` 的函数，如下所示：

```ts
function loggedMethod(originalMethod: any, _context: any) {
  function replacementMethod(this: any, ...args: any[]) {
    console.log('LOG: Entering method.')
    const result = originalMethod.call(this, ...args)
    console.log('LOG: Exiting method.')
    return result
  }

  return replacementMethod
}
```

“为什么到处都是 `any`？
难道这是 `any`Script！？”

稍安勿躁——这里为了保持简单，方便我们专注于该函数的作用。
请注意，`loggedMethod` 接收原始方法（`originalMethod`），并返回一个函数，该函数会：

1. 输出一条 "Entering..." 日志
2. 将 `this` 和所有参数传递给原始方法
3. 输出一条 "Exiting..." 日志
4. 返回原始方法返回的任何结果。

现在我们可以使用 `loggedMethod` 来*装饰*方法 `greet`：

```ts
class Person {
  name: string
  constructor(name: string) {
    this.name = name
  }

  @loggedMethod
  greet() {
    console.log(`Hello, my name is ${this.name}.`)
  }
}

const p = new Person('Ray')
p.greet()

// Output:
//
//   LOG: Entering method.
//   Hello, my name is Ray.
//   LOG: Exiting method.
```

我们刚刚在 `greet` 上方将 `loggedMethod` 作为装饰器使用——注意我们将其写为 `@loggedMethod`。
这样做时，它会被调用并接收方法*目标（target）*和一个*上下文对象（context object）*。
因为 `loggedMethod` 返回了一个新函数，所以该函数替换了 `greet` 的原始定义。

我们之前还没提到，`loggedMethod` 定义了第二个参数。
它被称为“上下文对象”，包含有关被装饰方法如何声明的一些有用信息——例如它是否是 `#private` 成员、是否为 `static`、或者方法的名称是什么。
让我们重写 `loggedMethod` 来利用这一点，并打印出被装饰方法的名称。

```ts
function loggedMethod(
  originalMethod: any,
  context: ClassMethodDecoratorContext,
) {
  const methodName = String(context.name)

  function replacementMethod(this: any, ...args: any[]) {
    console.log(`LOG: Entering method '${methodName}'.`)
    const result = originalMethod.call(this, ...args)
    console.log(`LOG: Exiting method '${methodName}'.`)
    return result
  }

  return replacementMethod
}
```

我们现在使用了 context 参数——它是 `loggedMethod` 中第一个类型严于 `any` 和 `any[]` 的地方。
TypeScript 提供了一个名为 `ClassMethodDecoratorContext` 的类型，用于为方法装饰器接收的上下文对象建模。

除了元数据之外，方法对应的上下文对象还提供了一个名为 `addInitializer` 的有用函数。
它提供了一种钩入构造函数开头（或者在处理 `static` 成员时钩入类自身初始化）的方法。

举个例子——在 JavaScript 中，经常会写出类似下面的代码模式：

```ts
class Person {
  name: string
  constructor(name: string) {
    this.name = name

    this.greet = this.greet.bind(this)
  }

  greet() {
    console.log(`Hello, my name is ${this.name}.`)
  }
}
```

或者，`greet` 也可以声明为一个初始化为箭头函数的属性：

```ts
class Person {
  name: string
  constructor(name: string) {
    this.name = name
  }

  greet = () => {
    console.log(`Hello, my name is ${this.name}.`)
  }
}
```

这样编写代码是为了确保当 `greet` 作为独立函数调用或作为回调传递时，`this` 不会被重新绑定：

```ts
const greet = new Person('Ray').greet

// We don't want this to fail!
greet()
```

我们可以编写一个装饰器，使用 `addInitializer` 在构造函数中为我们调用 `bind`：

```ts
function bound(originalMethod: any, context: ClassMethodDecoratorContext) {
  const methodName = context.name
  if (context.private) {
    throw new Error(
      `'bound' cannot decorate private properties like ${methodName as string}.`,
    )
  }
  context.addInitializer(function () {
    this[methodName] = this[methodName].bind(this)
  })
}
```

`bound` 没有返回任何内容——因此当它装饰一个方法时，它会保持原始方法不变。
相反，它会在任何其他字段初始化之前添加逻辑。

```ts
class Person {
  name: string
  constructor(name: string) {
    this.name = name
  }

  @bound
  @loggedMethod
  greet() {
    console.log(`Hello, my name is ${this.name}.`)
  }
}

const p = new Person('Ray')
const greet = p.greet

// Works!
greet()
```

注意，我们堆叠了两个装饰器——`@bound` 和 `@loggedMethod`。
这些装饰器以“相反的顺序”执行。
也就是说，`@loggedMethod` 装饰原始方法 `greet`，而 `@bound` 装饰 `@loggedMethod` 的结果。
在这个例子中，顺序并不重要——但如果你的装饰器存在副作用或依赖特定顺序，这就很重要了。

另外值得一提的是——如果你偏好这种代码风格，可以将这些装饰器写在同一行：

```ts
    @bound @loggedMethod greet() {
        console.log(`Hello, my name is ${this.name}.`);
    }
```

还有一点可能不那么显而易见：我们甚至可以编写*返回*装饰器函数的函数。
这使得对最终的装饰器进行定制变得可能。
如果我们愿意，可以让 `loggedMethod` 返回一个装饰器，并定制其记录日志的方式。

```ts
function loggedMethod(headMessage = 'LOG:') {
  return function actualDecorator(
    originalMethod: any,
    context: ClassMethodDecoratorContext,
  ) {
    const methodName = String(context.name)

    function replacementMethod(this: any, ...args: any[]) {
      console.log(`${headMessage} Entering method '${methodName}'.`)
      const result = originalMethod.call(this, ...args)
      console.log(`${headMessage} Exiting method '${methodName}'.`)
      return result
    }

    return replacementMethod
  }
}
```

如果这样做，我们需要先调用 `loggedMethod`，再将其作为装饰器使用。
然后，我们可以传入任意字符串作为打印到控制台的消息前缀。

```ts
class Person {
  name: string
  constructor(name: string) {
    this.name = name
  }

  @loggedMethod('⚠️')
  greet() {
    console.log(`Hello, my name is ${this.name}.`)
  }
}

const p = new Person('Ray')
p.greet()

// Output:
//
//   ⚠️ Entering method 'greet'.
//   Hello, my name is Ray.
//   ⚠️ Exiting method 'greet'.
```

装饰器不仅可以用于方法！
它们还可以用于属性/字段、getter、setter 以及自动访问器（auto-accessors）。
甚至类本身也可以被装饰，用于子类化和注册等操作。

要深入了解装饰器，可以阅读 [Axel Rauschmayer 的详尽总结](https://2ality.com/2022/10/javascript-decorators.html)。

有关所涉及变更的更多信息，可以[查看原始 Pull Request](https://github.com/microsoft/TypeScript/pull/50820)。

### 与旧版实验性装饰器的差异

如果你使用 TypeScript 已经有一段时间，可能已经知道它多年来一直支持“实验性”装饰器。
虽然这些实验性装饰器非常有用，但它们是以旧版装饰器提案为模型的，并且始终需要通过编译器标志 `--experimentalDecorators` 显式开启。
在没有此标志的情况下，任何在 TypeScript 中使用装饰器的尝试过去都会报错。

在可预见的未来，`--experimentalDecorators` 将继续存在；
但是，在未开启该标志的情况下，装饰器现在在所有新代码中都是合法的语法。
在未开启 `--experimentalDecorators` 时，它们的类型检查和代码生成方式会有所不同。
类型检查规则和代码生成的差异非常大，尽管装饰器*可以*编写为同时支持新旧两种装饰器行为，但现有的装饰器函数很可能无法做到这一点。

全新的装饰器提案与 `--emitDecoratorMetadata` 不兼容，且不允许装饰参数。
未来的 ECMAScript 提案可能会弥补这一差距。

最后需要指出的是：除了允许将装饰器放置在 `export` 关键字之前外，装饰器提案现在还支持将装饰器放置在 `export` 或 `export default` 之后。
唯一的例外是不允许混用这两种风格。

```js
// ✅ allowed
@register export default class Foo {
    // ...
}

// ✅ also allowed
export default @register class Bar {
    // ...
}

// ❌ error - before *and* after is not allowed
@before export @after class Bar {
    // ...
}
```

### 编写类型完备的装饰器

上述 `loggedMethod` 和 `bound` 装饰器示例故意保持了简单，省略了大量类型细节。

为装饰器编写类型可能相当复杂。
例如，上述 `loggedMethod` 的类型完备版本可能看起来像这样：

```ts
function loggedMethod<This, Args extends any[], Return>(
  target: (this: This, ...args: Args) => Return,
  context: ClassMethodDecoratorContext<
    This,
    (this: This, ...args: Args) => Return
  >,
) {
  const methodName = String(context.name)

  function replacementMethod(this: This, ...args: Args): Return {
    console.log(`LOG: Entering method '${methodName}'.`)
    const result = target.call(this, ...args)
    console.log(`LOG: Exiting method '${methodName}'.`)
    return result
  }

  return replacementMethod
}
```

我们必须使用类型参数 `This`、`Args` 和 `Return` 分别为原始方法的 `this` 类型、参数类型和返回值类型建模。

装饰器函数的具体定义复杂度取决于你希望保证什么。
请记住，装饰器的使用次数远多于编写次数，因此类型完备的版本通常更受青睐——但这显然需要在可读性之间做出权衡，因此请尽量保持简单。

未来将提供更多有关编写装饰器的文档——但[这篇文章](https://2ality.com/2022/10/javascript-decorators.html)已经详细介绍了装饰器的运行机制。

## `const` 类型参数

在推断对象的类型时，TypeScript 通常会选择一个更通用的类型。
例如在此示例中，`names` 的推断类型为 `string[]`：

```ts
type HasNames = { names: readonly string[] }
function getNamesExactly<T extends HasNames>(arg: T): T['names'] {
  return arg.names
}

// Inferred type: string[]
const names = getNamesExactly({ names: ['Alice', 'Bob', 'Eve'] })
```

这通常是为了允许后续进行修改。

然而，根据 `getNamesExactly` 的具体作用和使用方式，通常可能希望获得更具体的类型。

到目前为止，API 作者通常必须建议在某些地方添加 `as const` 来实现预期的类型推断：

```ts
// The type we wanted:
//    readonly ["Alice", "Bob", "Eve"]
// The type we got:
//    string[]
const names1 = getNamesExactly({ names: ['Alice', 'Bob', 'Eve'] })

// Correctly gets what we wanted:
//    readonly ["Alice", "Bob", "Eve"]
const names2 = getNamesExactly({ names: ['Alice', 'Bob', 'Eve'] } as const)
```

这样做既繁琐又容易遗忘。
在 TypeScript 5.0 中，你现在可以为类型参数声明添加 `const` 修饰符，使类似 `const` 的推断行为成为默认规则：

```ts
type HasNames = { names: readonly string[] }
function getNamesExactly<const T extends HasNames>(arg: T): T['names'] {
  //                       ^^^^^
  return arg.names
}

// Inferred type: readonly ["Alice", "Bob", "Eve"]
// Note: Didn't need to write 'as const' here
const names = getNamesExactly({ names: ['Alice', 'Bob', 'Eve'] })
```

请注意，`const` 修饰符并不*拒绝*可变值，也不要求不可变约束。
使用可变类型约束可能会产生出乎意料的结果。
例如：

```ts
declare function fnBad<const T extends string[]>(args: T): void

// 'T' is still 'string[]' since 'readonly ["a", "b", "c"]' is not assignable to 'string[]'
fnBad(['a', 'b', 'c'])
```

在这里，`T` 的推断候选类型是 `readonly ["a", "b", "c"]`，而只读数组不能用于需要可变数组的地方。
在这种情况下，类型推断会回退到约束条件，数组被视为 `string[]`，调用仍然可以顺利进行。

该函数更好的定义应该使用 `readonly string[]`：

```ts
declare function fnGood<const T extends readonly string[]>(args: T): void

// T is readonly ["a", "b", "c"]
fnGood(['a', 'b', 'c'])
```

同样需要牢记的是，`const` 修饰符仅影响在调用中直接编写的对象、数组和原始字面量表达式的推断，因此对于那些不会（或不能）通过 `as const` 修饰的参数，行为不会发生改变：

```ts
declare function fnGood<const T extends readonly string[]>(args: T): void
const arr = ['a', 'b', 'c']

// 'T' is still 'string[]'-- the 'const' modifier has no effect here
fnGood(arr)
```

有关更多详细信息，请[查看 Pull Request](https://github.com/microsoft/TypeScript/pull/51865) 以及引起该特性的动机 Issue（[第一篇](https://github.com/microsoft/TypeScript/issues/30680)与[第二篇](https://github.com/microsoft/TypeScript/issues/41114)）。

## 在 `extends` 中支持多个配置文件

在管理多个项目时，拥有一个供其他 `tsconfig.json` 文件继承的“基础”配置文件会非常有帮助。
这就是 TypeScript 支持 `extends` 字段以便从 `compilerOptions` 复制字段的原因。

```jsonc
// packages/front-end/src/tsconfig.json
{
  "extends": "../../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "../lib",
    // ...
  },
}
```

然而在某些场景下，你可能希望继承多个配置文件。
例如，假设你使用了[发布到 npm 的 TypeScript 基础配置文件](https://github.com/tsconfig/bases)。
如果你希望所有项目同时也使用来自 npm 上的 `@tsconfig/strictest` 包的配置项，那么有一个简单的解决方案：让 `tsconfig.base.json` 继承 `@tsconfig/strictest`：

```jsonc
// tsconfig.base.json
{
  "extends": "@tsconfig/strictest/tsconfig.json",
  "compilerOptions": {
    // ...
  },
}
```

这在一定程度上是可行的。
但如果你有任何项目*不*希望使用 `@tsconfig/strictest`，它们要么必须手动禁用这些选项，要么必须创建一个*不*继承 `@tsconfig/strictest` 的独立版本的 `tsconfig.base.json`。

为了提供更大的灵活性，TypeScript 5.0 现在允许 `extends` 字段接受多个配置项。
例如在此配置文件中：

```jsonc
{
  "extends": ["a", "b", "c"],
  "compilerOptions": {
    // ...
  },
}
```

这样写类似于直接继承 `c`，其中 `c` 继承 `b`，而 `b` 继承 `a`。
如果有任何字段发生“冲突”，则后面的配置项优先。

因此在以下示例中，最终的 `tsconfig.json` 会同时启用 `strictNullChecks` 和 `noImplicitAny`。

```jsonc
// tsconfig1.json
{
    "compilerOptions": {
        "strictNullChecks": true
    }
}

// tsconfig2.json
{
    "compilerOptions": {
        "noImplicitAny": true
    }
}

// tsconfig.json
{
    "extends": ["./tsconfig1.json", "./tsconfig2.json"],
    "files": ["./index.ts"]
}
```

再举一个例子，我们可以通过以下方式重写最初的示例：

```jsonc
// packages/front-end/src/tsconfig.json
{
  "extends": [
    "@tsconfig/strictest/tsconfig.json",
    "../../../tsconfig.base.json",
  ],
  "compilerOptions": {
    "outDir": "../lib",
    // ...
  },
}
```

更多详情请[阅读原始 Pull Request](https://github.com/microsoft/TypeScript/pull/50403)。

<!--

## Improved Type Argument Inference

TODO

## Improved `in` Checks Under `--noUncheckedIndexedAccess`

TODO

-->

## 所有 `enum` 都是联合 `enum`

当 TypeScript 最初引入枚举时，它们不过是一组具有相同类型的数值常量。

```ts
enum E {
  Foo = 10,
  Bar = 20,
}
```

`E.Foo` 和 `E.Bar` 唯一特殊之处在于它们可以赋值给任何期望 `E` 类型的变量。
除此之外，它们几乎就是普通的 `number`。

```ts
function takeValue(e: E) {}

takeValue(E.Foo) // works
takeValue(123) // error!
```

直到 TypeScript 2.0 引入枚举字面量类型，枚举才变得更加特殊。
枚举字面量类型赋予每个枚举成员专属的类型，并将枚举本身变成了每个成员类型的*联合（union）*。
它们还允许我们仅引用枚举类型的一个子集，并能够收窄（narrow）这些类型。

```ts
// Color is like a union of Red | Orange | Yellow | Green | Blue | Violet
enum Color {
  Red,
  Orange,
  Yellow,
  Green,
  Blue,
  /* Indigo, */ Violet,
}

// Each enum member has its own type that we can refer to!
type PrimaryColor = Color.Red | Color.Green | Color.Blue

function isPrimaryColor(c: Color): c is PrimaryColor {
  // Narrowing literal types can catch bugs.
  // TypeScript will error here because
  // we'll end up comparing 'Color.Red' to 'Color.Green'.
  // We meant to use ||, but accidentally wrote &&.
  return c === Color.Red && c === Color.Green && c === Color.Blue
}
```

为每个枚举成员赋予独立类型带来的一个问题是：这些类型在某种程度上与成员的实际值相关联。
在某些情况下，无法在编译时计算出该值——例如枚举成员可能是通过函数调用初始化的。

```ts
enum E {
  Blah = Math.random(),
}
```

每当遇到这些情况时，TypeScript 就会默默回退并采用旧的枚举策略。
这意味着放弃了联合和字面量类型的所有优势。

TypeScript 5.0 通过为每个计算成员创建唯一样式类型，成功使所有枚举都成为联合枚举。
这意味着现在所有枚举都可以被收窄，其成员也可以作为类型被引用。

有关此变更的更多详细信息，可以[在 GitHub 上阅读具体内容](https://github.com/microsoft/TypeScript/pull/50528)。

## `--moduleResolution bundler`

TypeScript 4.7 为 `--module` 和 `--moduleResolution` 设置引入了 `node16` 和 `nodenext` 选项。
这些选项的初衷是更准确地模拟 Node.js 中 ECMAScript 模块的具体查找规则；
然而该模式包含很多其他工具并不强制执行的限制。

例如，在 Node.js 的 ECMAScript 模块中，任何相对导入都必须包含文件扩展名：

```js
// entry.mjs
import * as utils from './utils' // ❌ wrong - we need to include the file extension.

import * as utils from './utils.mjs' // ✅ works
```

在 Node.js 和浏览器中存在这样做的理由——它加快了文件查找速度，并且对简易文件服务器更友好。
但对于许多使用打包工具（bundler）等工具的开发者来说，`node16`/`nodenext` 设置非常繁琐，因为打包工具并没有其中的大部分限制。
在某种程度上，原有的 `node` 解析模式对使用打包工具的人来说反而更合适。

但从某些方面来看，原始的 `node` 解析模式已经过时了。
大多数现代打包工具采用了 Node.js 中 ECMAScript 模块与 CommonJS 查找规则的融合机制。
例如，省略扩展名的导入可以像 CommonJS 一样正常工作；但在查找包的 [`export` 条件](https://nodejs.org/api/packages.html#nested-conditions)时，它们又会像 ECMAScript 文件一样优先匹配 `import` 条件。

为了模拟打包工具的工作方式，TypeScript 现在引入了一种新策略：`--moduleResolution bundler`。

```jsonc
{
  "compilerOptions": {
    "target": "esnext",
    "moduleResolution": "bundler",
  },
}
```

如果你正在使用 Vite、esbuild、swc、Webpack、Parcel 等实现了混合查找策略的现代打包工具，新的 `bundler` 选项会非常契合你的需求。

另一方面，如果你正在编写打算发布到 npm 的库，使用 `bundler` 选项可能会掩盖那些*没有*使用打包工具的用户可能遇到的兼容性问题。
因此在这些情况下，使用 `node16` 或 `nodenext` 解析选项通常是更好的选择。

欲了解有关 `--moduleResolution bundler` 的更多信息，请[查看实现该特性的 Pull Request](https://github.com/microsoft/TypeScript/pull/51669)。

## 模块解析自定义标志

JavaScript 工具现在可能会模拟“混合”解析规则，就像我们在上述 `bundler` 模式中所描述的那样。
由于不同工具的支持程度可能略有差异，TypeScript 5.0 提供了启用或禁用若干特性的方式，以适配你的具体配置。

### `allowImportingTsExtensions`

`--allowImportingTsExtensions` 允许 TypeScript 文件使用 TypeScript 特有的扩展名（如 `.ts`、`.mts` 或 `.tsx`）相互导入。

该标志仅在启用 `--noEmit` 或 `--emitDeclarationOnly` 时才允许使用，因为这些导入路径在 JavaScript 输出文件中无法在运行时解析。
这里的预期是你的解析器（如打包工具、运行时或其他工具）能够处理 `.ts` 文件之间的此类导入。

### `resolvePackageJsonExports`

`--resolvePackageJsonExports` 强制 TypeScript 在从 `node_modules` 中读取包时参考 [`package.json` 文件的 `exports` 字段](https://nodejs.org/api/packages.html#exports)。

在 `--moduleResolution` 设为 `node16`、`nodenext` 和 `bundler` 时，该选项默认为 `true`。

### `resolvePackageJsonImports`

当从其祖先目录包含 `package.json` 的文件中执行以 `#` 开头的查找时，`--resolvePackageJsonImports` 强制 TypeScript 参考 [`package.json` 文件的 `imports` 字段](https://nodejs.org/api/packages.html#imports)。

在 `--moduleResolution` 设为 `node16`、`nodenext` 和 `bundler` 时，该选项默认为 `true`。

### `allowArbitraryExtensions`

在 TypeScript 5.0 中，当导入路径以非已知 JavaScript 或 TypeScript 文件扩展名结尾时，编译器将按 `{file basename}.d.{extension}.ts` 的格式查找该路径的声明文件。
例如，如果你在打包工具项目中使用 CSS 加载器，你可能希望为这些样式表编写（或生成）声明文件：

```css
/* app.css */
.cookie-banner {
  display: none;
}
```

```ts
// app.d.css.ts
declare const css: {
  cookieBanner: string
}
export default css
```

```ts
// App.tsx
import styles from './app.css'

styles.cookieBanner // string
```

默认情况下，此导入将抛出错误，提示 TypeScript 无法识别此文件类型，且你的运行时可能不支持导入它。
但如果你已配置运行时或打包工具对其进行处理，则可以使用新的 `--allowArbitraryExtensions` 编译器选项来忽略该错误。

需要注意的是，在过去通常可以通过添加名为 `app.css.d.ts` 而非 `app.d.css.ts` 的声明文件来实现类似的效果——然而这只是通过 Node 针对 CommonJS 的 `require` 解析规则起作用的。
严格来说，前者被解释为名为 `app.css.js` 的 JavaScript 文件的声明文件。
因为在 Node 的 ESM 支持中相对文件导入必须包含扩展名，所以在 `--moduleResolution node16` 或 `nodenext` 下的 ESM 文件中，TypeScript 会针对我们的示例报错。

有关更多信息，请阅读[该特性的提案](https://github.com/microsoft/TypeScript/issues/50133)以及[对应的 Pull Request](https://github.com/microsoft/TypeScript/pull/51435)。

### `customConditions`

`--customConditions` 接受一个额外的[条件（conditions）](https://nodejs.org/api/packages.html#nested-conditions)列表，当 TypeScript 从 `package.json` 的 [`exports`](https://nodejs.org/api/packages.html#exports) 或 [`imports`](https://nodejs.org/api/packages.html#imports) 字段进行解析时应满足这些条件。
这些条件会添加到解析器默认使用的任何现有条件中。

例如，在 `tsconfig.json` 中设置此字段如下：

```jsonc
{
  "compilerOptions": {
    "target": "es2022",
    "moduleResolution": "bundler",
    "customConditions": ["my-condition"],
  },
}
```

只要在 `package.json` 中引用了 `exports` 或 `imports` 字段，TypeScript 就会考虑名为 `my-condition` 的条件。

因此，当从包含以下 `package.json` 的包中导入时：

```jsonc
{
  // ...
  "exports": {
    ".": {
      "my-condition": "./foo.mjs",
      "node": "./bar.mjs",
      "import": "./baz.mjs",
      "require": "./biz.mjs",
    },
  },
}
```

TypeScript 将尝试查找与 `foo.mjs` 对应的文件。

该字段仅在 `--moduleResolution` 选项为 `node16`、`nodenext` 和 `bundler` 时有效。

## `--verbatimModuleSyntax`

默认情况下，TypeScript 会执行名为*导入擦除（import elision）*的操作。
简单来说，如果你编写了类似以下的代码：

```ts
import { Car } from './car'

export function drive(car: Car) {
  // ...
}
```

TypeScript 会检测到你仅将导入用于类型，并完全丢弃该导入。
生成的 JavaScript 代码可能看起来像这样：

```js
export function drive(car) {
  // ...
}
```

大多数情况下这很好，因为如果 `Car` 不是从 `./car` 导出的值，我们就会遇到运行时错误。

但在某些边缘情况下，这确实增加了一层复杂性。
例如，请注意这里没有类似 `import "./car";` 的语句——该导入被完全丢弃了。
对于带有或不带有副作用的模块，这实际上会带来行为差异。

TypeScript 输出 JavaScript 的生成策略还具有另外几层复杂性——导入擦除并不总是只取决于导入的使用方式，它通常还会参考值的声明方式。
因此，类似以下的代码：

```ts
export { Car } from './car'
```

究竟应该保留还是丢弃，并不总是那么明确。
如果 `Car` 是使用 `class` 之类声明的，那么它可以在生成的 JavaScript 文件中保留。
但如果 `Car` 仅声明为 `type` 别名或 `interface`，那么 JavaScript 文件根本不应该导出 `Car`。

虽然 TypeScript 或许能够根据跨文件的信息做出这些代码生成决策，但并非每个编译器都能做到。

在导入和导出中使用 `type` 修饰符在一定程度上有助于解决这些情况。
我们可以使用 `type` 修饰符明确标明某个导入或导出仅用于类型分析，并可以在 JavaScript 文件中完全丢弃：

```ts
// This statement can be dropped entirely in JS output
import type * as car from './car'

// The named import/export 'Car' can be dropped in JS output
import { type Car } from './car'
export { type Car } from './car'
```

`type` 修饰符单独使用时作用有限——默认情况下模块擦除仍然会丢弃导入，而且没有任何机制强制你区分 `type` 与普通导入/导出。
因此，TypeScript 提供了 `--importsNotUsedAsValues` 标志以确保你使用 `type` 修饰符，提供了 `--preserveValueImports` 以防止*某些*模块擦除行为，并提供了 `--isolatedModules` 以确保你的 TypeScript 代码可以在不同的编译器间正常工作。
遗憾的是，理解这 3 个标志的细节非常困难，并且仍然存在一些会出现意外行为的边缘情况。

TypeScript 5.0 引入了一个名为 `--verbatimModuleSyntax` 的新选项来简化这一局面。
其规则要简单得多——任何没有 `type` 修饰符的导入或导出都会被原样保留。
任何使用了 `type` 修饰符的内容都会被完全丢弃。

```ts
// Erased away entirely.
import type { A } from 'a'

// Rewritten to 'import { b } from "bcd";'
import { b, type c, type d } from 'bcd'

// Rewritten to 'import {} from "xyz";'
import { type xyz } from 'xyz'
```

使用这个新选项后，所见即所得。

不过在模块互操作性方面，这确实会产生一些影响。
在该标志下，当你的设置或文件扩展名暗指不同的模块系统时，ECMAScript 的 `import` 和 `export` 不会被重写为 `require` 调用。
相反，你会收到一个错误。
如果你需要生成使用 `require` 和 `module.exports` 的代码，你必须使用早于 ES2015 的 TypeScript 模块语法：

<table>
<thead>
    <tr>
        <th>输入 TypeScript</th>
        <th>输出 JavaScript</th>
    </tr>
</thead>

<tr>
<td>

```ts
import foo = require('foo')
```

</td>
<td>

```js
const foo = require('foo')
```

</td>
</tr>
<tr>
<td>

```ts
function foo() {}
function bar() {}
function baz() {}

export = {
  foo,
  bar,
  baz,
}
```

</td>
<td>

```js
function foo() {}
function bar() {}
function baz() {}

module.exports = {
  foo,
  bar,
  baz,
}
```

</td>
</tr>
</table>

虽然这是一项限制，但它确实有助于使某些问题变得更加清晰。
例如，在 `--module node16` 下很容易忘记设置 [`package.json` 中的 `type` 字段](https://nodejs.org/api/packages.html#type)。
结果，开发者在不知不觉中开始编写 CommonJS 模块而不是 ES 模块，从而导致意外的查找规则和 JavaScript 输出。
这个新标志确保你对自己使用的文件类型有清晰的预期，因为两者语法本就有所不同。

由于 `--verbatimModuleSyntax` 提供了比 `--importsNotUsedAsValues` 和 `--preserveValueImports` 更一致的体验，这两个现有的标志已被弃用，推荐使用新选项。

更多详情请阅读[原始 Pull Request](https://github.com/microsoft/TypeScript/pull/52203)及其[提案 Issue](https://github.com/microsoft/TypeScript/issues/51479)。

## 支持 `export type *`

当 TypeScript 3.8 引入仅类型导入（type-only imports）时，这种新语法不允许用于 `export * from "module"` 或 `export * as ns from "module"` 重新导出。TypeScript 5.0 增加了对这两种形式的支持：

```ts
// models/vehicles.ts
export class Spaceship {
  // ...
}

// models/index.ts
export type * as vehicles from './vehicles'

// main.ts
import { vehicles } from './models'

function takeASpaceship(s: vehicles.Spaceship) {
  // ✅ ok - `vehicles` only used in a type position
}

function makeASpaceship() {
  return new vehicles.Spaceship()
  //         ^^^^^^^^
  // 'vehicles' cannot be used as a value because it was exported using 'export type'.
}
```

你可以[在此阅读有关实现的更多信息](https://github.com/microsoft/TypeScript/pull/52217)。

## JSDoc 支持 `@satisfies`

TypeScript 4.9 引入了 `satisfies` 运算符。
它确保表达式的类型兼容，同时不影响类型本身。
例如，看以下代码：

```ts
interface CompilerOptions {
  strict?: boolean
  outDir?: string
  // ...
}

interface ConfigSettings {
  compilerOptions?: CompilerOptions
  extends?: string | string[]
  // ...
}

let myConfigSettings = {
  compilerOptions: {
    strict: true,
    outDir: '../lib',
    // ...
  },

  extends: ['@tsconfig/strictest/tsconfig.json', '../../../tsconfig.base.json'],
} satisfies ConfigSettings
```

在这里，TypeScript 知道 `myConfigSettings.extends` 声明为一个数组——因为虽然 `satisfies` 验证了对象的类型，但它并没有粗暴地将其转换为 `CompilerOptions` 并丢失信息。
因此，如果我们想要对 `extends` 调用 map，完全没有问题：

```ts
declare function resolveConfig(configPath: string): CompilerOptions

let inheritedConfigs = myConfigSettings.extends.map(resolveConfig)
```

这对 TypeScript 用户很有帮助，但许多人使用带有 JSDoc 注解的 TypeScript 来对 JavaScript 代码进行类型检查。
这就是为什么 TypeScript 5.0 支持了一个名为 `@satisfies` 的新 JSDoc 标签，其功能完全相同。

`/** @satisfies */` 可以捕获类型不匹配：

```js
// @ts-check

/**
 * @typedef CompilerOptions
 * @prop {boolean} [strict]
 * @prop {string} [outDir]
 */

/**
 * @satisfies {CompilerOptions}
 */
let myCompilerOptions = {
  outdir: '../lib',
  //  ~~~~~~ oops! we meant outDir
}
```

但它会保留表达式的原始类型，使我们稍后在代码中可以更精确地使用这些值：

```js
// @ts-check

/**
 * @typedef CompilerOptions
 * @prop {boolean} [strict]
 * @prop {string} [outDir]
 */

/**
 * @typedef ConfigSettings
 * @prop {CompilerOptions} [compilerOptions]
 * @prop {string | string[]} [extends]
 */

/**
 * @satisfies {ConfigSettings}
 */
let myConfigSettings = {
  compilerOptions: {
    strict: true,
    outDir: '../lib',
  },
  extends: ['@tsconfig/strictest/tsconfig.json', '../../../tsconfig.base.json'],
}

let inheritedConfigs = myConfigSettings.extends.map(resolveConfig)
```

`/** @satisfies */` 也可以内联用于任何带括号的表达式中。
我们可以将 `myConfigSettings` 写成这样：

```ts
let myConfigSettings = /** @satisfies {ConfigSettings} */ {
  compilerOptions: {
    strict: true,
    outDir: '../lib',
  },
  extends: ['@tsconfig/strictest/tsconfig.json', '../../../tsconfig.base.json'],
}
```

为什么这么做？
好吧，当你处于更深层的代码中（例如函数调用内部）时，这通常更有意义：

```js
compileCode(
  /** @satisfies {CompilerOptions} */ (
    {
      // ...
    }
  ),
)
```

感谢 [Oleksandr Tarasiuk](https://github.com/a-tarasyuk) 提供了[该特性](https://github.com/microsoft/TypeScript/pull/51753)！

## JSDoc 支持 `@overload`

在 TypeScript 中，你可以为函数指定重载。
重载使我们能够声明一个函数可以使用不同的参数调用，并可能返回不同的结果。
它们可以限制调用方实际使用函数的方式，并细化返回的结果：

```ts
// Our overloads:
function printValue(str: string): void
function printValue(num: number, maxFractionDigits?: number): void

// Our implementation:
function printValue(value: string | number, maximumFractionDigits?: number) {
  if (typeof value === 'number') {
    const formatter = Intl.NumberFormat('en-US', {
      maximumFractionDigits,
    })
    value = formatter.format(value)
  }

  console.log(value)
}
```

在这里，我们声明了 `printValue` 的第一个参数既可以是 `string` 也可以是 `number`。
如果传入的是 `number`，它还可以接收第二个参数来决定可以打印多少位小数。

TypeScript 5.0 现在允许 JSDoc 使用新的 `@overload` 标签声明重载。
每个带有 `@overload` 标签的 JSDoc 注释都会被视为后续函数声明的独立重载：

```js
// @ts-check

/**
 * @overload
 * @param {string} value
 * @return {void}
 */

/**
 * @overload
 * @param {number} value
 * @param {number} [maximumFractionDigits]
 * @return {void}
 */

/**
 * @param {string | number} value
 * @param {number} [maximumFractionDigits]
 */
function printValue(value, maximumFractionDigits) {
  if (typeof value === 'number') {
    const formatter = Intl.NumberFormat('en-US', {
      maximumFractionDigits,
    })
    value = formatter.format(value)
  }

  console.log(value)
}
```

现在，无论我们是在 TypeScript 还是 JavaScript 文件中编写代码，如果我们调用函数的方式不正确，TypeScript 都能向我们报错：

```ts
// all allowed
printValue('hello!')
printValue(123.45)
printValue(123.45, 2)

printValue('hello!', 123) // error!
```

感谢 [Tomasz Lenarcik](https://github.com/apendua)[实现](https://github.com/microsoft/TypeScript/pull/51234)了这一全新标签。

## 在 `--build` 模式下传递特定于代码生成的标志

TypeScript 现在允许在 `--build` 模式下传递以下标志：

- `--declaration`
- `--emitDeclarationOnly`
- `--declarationMap`
- `--sourceMap`
- `--inlineSourceMap`

这使得在开发构建与生产构建不同的场景下定制构建过程变得容易得多。

例如，库的开发构建可能不需要生成声明文件，但生产构建需要。
项目可以配置默认关闭声明生成，只需使用以下命令进行构建：

```sh
tsc --build -p ./my-project-dir
```

当你在内循环中完成迭代后，“生产”构建只需传递 `--declaration` 标志即可：

```sh
tsc --build -p ./my-project-dir --declaration
```

有关此变更的[更多信息请参见此处](https://github.com/microsoft/TypeScript/pull/51241)。

## 编辑器中不区分大小写的导入排序

在 Visual Studio 和 VS Code 等编辑器中，TypeScript 为整理和排序导入/导出提供了支持。
然而，对于列表何时算作“已排序”，通常会有不同的理解。

例如，以下导入列表是否已排序？

```ts
import { Toggle, freeze, toBoolean } from './utils'
```

令人惊讶的是，答案可能是“视情况而定”。
如果我们*不*关心大小写敏感性，那么这个列表显然没有排序。
字母 `f` 排在 `t` 和 `T` 之前。

但在大多数编程语言中，排序默认是比较字符串的字节值。
JavaScript 比较字符串的方式意味着 `"Toggle"` 始终排在 `"freeze"` 之前，因为根据 [ASCII 字符编码](https://en.wikipedia.org/wiki/ASCII)，大写字母排在小写字母之前。
因此从这个角度来看，该导入列表是已排序的。

TypeScript 此前认为该导入列表已排序，因为它执行的是基本的大小写敏感排序。
对于偏好大小写*不敏感*排序、或者使用默认要求大小写不敏感排序的 ESLint 等工具的开发者来说，这可能会让人感到沮丧。

TypeScript 现在默认自动检测大小写敏感性。
这意味着 TypeScript 与 ESLint 等工具通常不会在如何最好地对导入进行排序上产生“冲突”。

我们的团队还在试验[更多的排序策略，你可以在这里了解](https://github.com/microsoft/TypeScript/pull/52115)。
这些选项最终可能会通过编辑器进行配置。
目前它们仍处于不稳定和实验阶段，今天你已经可以在 VS Code 中通过 JSON 配置中的 `typescript.unstable` 入口来体验它们。
以下是你可以尝试的所有选项（均设置为默认值）：

```jsonc
{
  "typescript.unstable": {
    // Should sorting be case-sensitive? Can be:
    // - true
    // - false
    // - "auto" (auto-detect)
    "organizeImportsIgnoreCase": "auto",

    // Should sorting be "ordinal" and use code points or consider Unicode rules? Can be:
    // - "ordinal"
    // - "unicode"
    "organizeImportsCollation": "ordinal",

    // Under `"organizeImportsCollation": "unicode"`,
    // what is the current locale? Can be:
    // - [any other locale code]
    // - "auto" (use the editor's locale)
    "organizeImportsLocale": "en",

    // Under `"organizeImportsCollation": "unicode"`,
    // should upper-case letters or lower-case letters come first? Can be:
    // - false (locale-specific)
    // - "upper"
    // - "lower"
    "organizeImportsCaseFirst": false,

    // Under `"organizeImportsCollation": "unicode"`,
    // do runs of numbers get compared numerically (i.e. "a1" < "a2" < "a100")? Can be:
    // - true
    // - false
    "organizeImportsNumericCollation": true,

    // Under `"organizeImportsCollation": "unicode"`,
    // do letters with accent marks/diacritics get sorted distinctly
    // from their "base" letter (i.e. is é different from e)? Can be
    // - true
    // - false
    "organizeImportsAccentCollation": true,
  },
  "javascript.unstable": {
    // same options valid here...
  },
}
```

你可以阅读有关[自动检测并指定大小写敏感性的原始工作](https://github.com/microsoft/TypeScript/pull/51733)，以及随后的[更广泛选项集](https://github.com/microsoft/TypeScript/pull/52115)的更多详细信息。

## 穷尽式 `switch`/`case` 补全

编写 `switch` 语句时，TypeScript 现在会检测被检查的值是否具有字面量类型。
如果是，它将提供代码补全，自动为每个未覆盖的 `case` 生成脚手架代码。

![基于字面量类型通过自动补全生成的一组 `case` 语句。](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2023/01/switchCaseSnippets-5-0_1.gif)

你可以[在 GitHub 上查看实现的具体细节](https://github.com/microsoft/TypeScript/pull/50996)。

## 速度、内存和包体积优化

TypeScript 5.0 在代码结构、数据结构和算法实现方面进行了大量强有力的改进。
这一切意味着你的整体体验都将更快——不仅运行 TypeScript 更快，连安装它都更快。

以下是我们记录的相对于 TypeScript 4.9 在速度和体积方面的一些显著提升：

| 场景                        | 相比 TS 4.9 的时间或体积 |
| --------------------------- | ------------------------ |
| material-ui 构建时间        | 89%                      |
| TypeScript 编译器启动时间   | 89%                      |
| Playwright 构建时间         | 88%                      |
| TypeScript 编译器自构建时间 | 87%                      |
| Outlook Web 构建时间        | 82%                      |
| VS Code 构建时间            | 80%                      |
| typescript npm 包体积       | 59%                      |

![TypeScript 5.0 相比 TypeScript 4.9 的构建/运行时间及包体积图表：material-ui 文档构建时间：89%；Playwright 构建时间：88%；tsc 启动时间：87%；tsc 构建时间：87%；Outlook Web 构建时间：82%；VS Code 构建时间：80%；typescript 包体积：59%](https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2023/03/speed-and-size-5-0-rc.png?1)

如何做到的？
我们未来会详细介绍几项值得注意的改进。
不过我们不会让你苦等那篇博客文章。

首先，我们最近将 TypeScript 从命名空间（namespaces）迁移到了模块（modules），这使我们能够利用现代构建工具来执行诸如作用域提升（scope hoisting）等优化。
借助这些工具、重新审视打包策略并移除部分已弃用的代码，我们将 TypeScript 4.9 的 63.8 MB 包体积缩减了约 26.4 MB。
它还通过直接函数调用带来了显著的速度提升。

TypeScript 还增强了编译器内部对象类型的一致性（uniformity），并精简了其中一些对象类型所存储的数据。
这减少了多态（polymorphic）和超多态（megamorphic）调用点，同时抵消了统一对象形态所需的大部分必要内存消耗。

我们在将信息序列化为字符串时还执行了一些缓存操作。
类型展示（可能发生在错误报告、声明生成、代码补全等环节中）最终开销相当大。
TypeScript 现在缓存了一些常用机制，以便在这些操作之间复用。

我们对解析器进行的另一项显著改进是利用 `var` 偶尔规避跨闭包使用 `let` 和 `const` 的开销。
这提高了一部分解析性能。

总体而言，我们预期大多数代码库在 TypeScript 5.0 中都会看到速度提升，并且能够稳定复现 10% 到 20% 的性能提升。
当然，具体情况取决于硬件和代码库特征，但我们鼓励你立即在自己的代码库中尝试！

有关更多信息，请参阅我们的一些显著优化：

- [迁移到模块（Migrate to Modules）](https://github.com/microsoft/TypeScript/pull/51387)
- [`Node` 单态化（`Node` Monomorphization）](https://github.com/microsoft/TypeScript/pull/51682)
- [`Symbol` 单态化（`Symbol` Monomorphization）](https://github.com/microsoft/TypeScript/pull/51880)
- [`Identifier` 体积缩减（`Identifier` Size Reduction）](https://github.com/microsoft/TypeScript/pull/52170)
- [`Printer` 缓存机制（`Printer` Caching）](https://github.com/microsoft/TypeScript/pull/52382)
- [有限使用 `var`（Limited Usage of `var`）](https://github.com/microsoft/TypeScript/issues/52924)

## 破坏性变更与弃用项

### 运行时要求

TypeScript 现在以 ECMAScript 2018 为目标。
对于 Node 用户，这意味着最低版本要求至少为 Node.js 10 及更高版本。

### `lib.d.ts` 变更

DOM 类型的生成方式发生变更可能会对现有代码产生影响。
值得注意的是，某些属性已从 `number` 转换为数值字面量类型，用于剪切、复制和粘贴事件处理的属性和方法已跨接口移动。

### API 破坏性变更

在 TypeScript 5.0 中，我们迁移到了模块结构，移除了一些不必要的接口，并进行了一些正确性改进。
有关变更的更多详细信息，请参阅我们的 [API 破坏性变更](https://github.com/microsoft/TypeScript/wiki/API-Breaking-Changes)页面。

### 关系运算符中禁止隐式类型转换

如果你编写可能导致隐式字符串到数字转换的代码，TypeScript 中的某些运算此前已经会发出警告：

```ts
function func(ns: number | string) {
  return ns * 4 // Error, possible implicit coercion
}
```

在 5.0 中，这同样适用于关系运算符 `>`, `<`, `<=`, 和 `>=`：

```ts
function func(ns: number | string) {
  return ns > 4 // Now also an error
}
```

如有需要，可以使用 `+` 显式将操作数转换为 `number`：

```ts
function func(ns: number | string) {
  return +ns > 4 // OK
}
```

该[正确性改进](https://github.com/microsoft/TypeScript/pull/52048)由 [Mateusz Burzyński](https://github.com/Andarist) 贡献。

### 枚举机制重构

自第一个版本发布以来，TypeScript 的 `enum` 就一直存在一些长期存在的怪异行为。
在 5.0 中，我们清理了其中一些问题，并减少了理解可声明的各种 `enum` 所需的概念数量。

作为其中的一部分，你可能会看到两个主要的新错误。
首先，将超出域范围的字面量赋值给 `enum` 类型现在会按预期报错：

```ts
enum SomeEvenDigit {
  Zero = 0,
  Two = 2,
  Four = 4,
}

// Now correctly an error
let m: SomeEvenDigit = 1
```

另一个问题是，声明某些间接混合字符串/数字的 `enum` 形式会错误地创建全数字的 `enum`：

```ts
enum Letters {
  A = 'a',
}
enum Numbers {
  one = 1,
  two = Letters.A,
}

// Now correctly an error
const t: number = Numbers.two
```

你可以[查看相关变更了解更多详情](https://github.com/microsoft/TypeScript/pull/50528)。

### 在 `--experimentalDecorators` 下对构造函数参数装饰器进行更准确的类型检查

TypeScript 5.0 让 `--experimentalDecorators` 下的装饰器类型检查更加准确。
一个显而易见的地方是在构造函数参数上使用装饰器时：

```ts
export declare const inject: (
  entity: any,
) => (target: object, key: string | symbol, index?: number) => void

export class Foo {}

export class C {
  constructor(@inject(Foo) private x: any) {}
}
```

该调用将失败，因为 `key` 期望是 `string | symbol`，但构造函数参数接收到的 key 为 `undefined`。
正确的修复方法是更改 `inject` 中 `key` 的类型。
如果你使用的库无法升级，一个合理的变通方法是将 `inject` 包装在一个更类型安全的装饰器函数中，并在 `key` 上使用类型断言。

更多详情请[参见此 Issue](https://github.com/microsoft/TypeScript/issues/52435)。

### 弃用项与默认值变更

在 TypeScript 5.0 中，我们弃用了以下设置及设置值：

- `--target: ES3`
- `--out`
- `--noImplicitUseStrict`
- `--keyofStringsOnly`
- `--suppressExcessPropertyErrors`
- `--suppressImplicitAnyIndexErrors`
- `--noStrictGenericChecks`
- `--charset`
- `--importsNotUsedAsValues`
- `--preserveValueImports`
- 项目引用中的 `prepend`（`prepend` in project references）

这些配置在 TypeScript 5.5 之前仍将允许使用，届时它们将被彻底移除；不过如果你正在使用这些设置，将会收到警告。
在 TypeScript 5.0 以及未来的 5.1、5.2、5.3 和 5.4 版本中，你可以指定 `"ignoreDeprecations": "5.0"` 来消除这些警告。
我们很快还会发布 4.9 补丁版本，以允许指定 `ignoreDeprecations`，从而实现更平滑的升级。
除了弃用项之外，我们还更改了一些设置，以更好地改进 TypeScript 的跨平台行为。

`--newLine` 控制输出的 JavaScript 文件中的换行符，如果未指定，过去会根据当前操作系统进行推断。
我们认为构建应该尽可能具有确定性，而且 Windows 记事本现在也支持换行符（LF），因此新的默认设置为 `LF`。
原先针对特定操作系统的推断行为不再提供。

`--forceConsistentCasingInFileNames` 确保项目中对同一文件名的所有引用在大小写上保持一致，现在的默认值为 `true`。
这有助于发现在不区分大小写的文件系统上编写代码时的大小写差异问题。

你可以留下反馈，并在 [5.0 弃用项追踪 Issue](https://github.com/microsoft/TypeScript/issues/51909) 中查看更多信息。
