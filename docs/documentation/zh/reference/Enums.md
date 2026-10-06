---
title: 枚举
layout: docs
permalink: /zh/docs/handbook/enums.html
oneline: TypeScript 枚举的工作原理
handbook: 'true'
---

枚举（Enums）是 TypeScript 中为数不多的几个不是 JavaScript 类型层面扩展的特性之一。

枚举允许开发者定义一组具名常量。
使用枚举可以更轻松地表达意图，或创建一组互不相同的分支情况。
TypeScript 同时支持基于数字和基于字符串的枚举。

## 数字枚举

我们首先从数字枚举开始介绍，如果你有其他编程语言的背景，可能会对它更为熟悉。
可以使用 `enum` 关键字来定义枚举。

```ts twoslash
enum Direction {
  Up = 1,
  Down,
  Left,
  Right,
}
```

在上面的代码中，我们定义了一个数字枚举，其中 `Up` 初始化为 `1`。
紧随其后的所有成员都会从该值开始自动自增。
换句话说，`Direction.Up` 的值为 `1`，`Down` 为 `2`，`Left` 为 `3`，`Right` 为 `4`。

如果需要，我们也可以完全省略初始化器：

```ts twoslash
enum Direction {
  Up,
  Down,
  Left,
  Right,
}
```

此时，`Up` 的值将为 `0`，`Down` 为 `1`，依此类推。
这种自动自增的行为非常适合那些我们不在乎成员具体的值是多少，但要求同一枚举中每个值都互不相同的场景。

使用枚举非常简单：只需像访问对象属性一样访问枚举的任意成员，并使用枚举名来声明类型：

```ts twoslash
enum UserResponse {
  No = 0,
  Yes = 1,
}

function respond(recipient: string, message: UserResponse): void {
  // ...
}

respond('Princess Caroline', UserResponse.Yes)
```

数字枚举可以混入[计算成员和常量成员（见下文）](#computed-and-constant-members)。
简而言之，没有初始化器的枚举成员要么必须排在第一位，要么必须排在使用数值常量或其他常量枚举成员初始化的数字枚举成员之后。
换句话说，以下写法是不被允许的：

```ts twoslash
// @errors: 1061
const getSomeValue = () => 23
// ---cut---
enum E {
  A = getSomeValue(),
  B,
}
```

## 字符串枚举

字符串枚举是一个类似的概念，但在[运行时存在一些细微差异](#enums-at-runtime)，如下文所述。
在字符串枚举中，每个成员都必须使用字符串字面量或另一个字符串枚举成员进行常量初始化。

```ts twoslash
enum Direction {
  Up = 'UP',
  Down = 'DOWN',
  Left = 'LEFT',
  Right = 'RIGHT',
}
```

虽然字符串枚举没有自动自增的行为，但字符串枚举的优势在于它们具有良好的“可序列化性”（serialize）。
换言之，如果你在调试时需要读取数字枚举的运行时值，该值通常是不透明的——它本身无法传达任何有用的含义（尽管[反向映射](#reverse-mappings)通常有所帮助）。而字符串枚举允许你在代码运行时提供有意义且可读的值，且该值独立于枚举成员本身的名称。

## 异构枚举（Heterogeneous enums）

从技术上讲，枚举可以混合包含字符串和数字成员，但通常并不清楚为什么要这样做：

```ts twoslash
enum BooleanLikeHeterogeneousEnum {
  No = 0,
  Yes = 'YES',
}
```

除非你确实想以某种巧妙的方式利用 JavaScript 的运行时行为，否则建议不要这样做。

## 计算成员和常量成员

每个枚举成员都有一个与之关联的值，该值可以是*常量*（constant）或*计算得出的*（computed）。
在以下情况下，枚举成员被视为常量：

- 它是枚举中的第一个成员且没有初始化器，此时它被赋值为 `0`：

  ```ts twoslash
  // E.X is constant:
  enum E {
    X,
  }
  ```

- 它没有初始化器，且前一个枚举成员是*数字*常量。
  此时当前枚举成员的值将是前一个枚举成员的值加一。

  ```ts twoslash
  // All enum members in 'E1' and 'E2' are constant.

  enum E1 {
    X,
    Y,
    Z,
  }

  enum E2 {
    A = 1,
    B,
    C,
  }
  ```

- 枚举成员使用常量枚举表达式进行初始化。
  常量枚举表达式是 TypeScript 表达式的一个子集，可以在编译时完全求值。
  满足以下条件的表达式即为常量枚举表达式：
  1. 字面量枚举表达式（基本上就是字符串字面量或数字字面量）
  2. 对先前定义的常量枚举成员的引用（可以来自不同的枚举）
  3. 带括号的常量枚举表达式
  4. 应用于常量枚举表达式的 `+`、`-`、`~` 一元运算符之一
  5. 以常量枚举表达式作为操作数的 `+`、`-`、`*`、`/`、`%`、`<<`、`>>`、`>>>`、`&`、`|`、`^` 二元运算符

  如果常量枚举表达式求值结果为 `NaN` 或 `Infinity`，则会在编译时报错。

在所有其他情况下，枚举成员都被视为计算成员。

```ts twoslash
enum FileAccess {
  // constant members
  None,
  Read = 1 << 1,
  Write = 1 << 2,
  ReadWrite = Read | Write,
  // computed member
  G = '123'.length,
}
```

## 联合枚举与枚举成员类型

常量枚举成员中存在一个无需计算的特殊子集：字面量枚举成员（literal enum members）。
字面量枚举成员是指没有初始值，或者其初始值符合以下条件的常量枚举成员：

- 任何字符串字面量（例如 `"foo"`、`"bar"`、`"baz"`）
- 任何数字字面量（例如 `1`、`100`）
- 应用于任何数字字面量的一元减号（例如 `-1`、`-100`）

当枚举中的所有成员都具有字面量枚举值时，就会触发一些特殊的语义。

首先，枚举成员本身也可以作为类型使用！
例如，我们可以指定某些成员*只能*拥有某个特定枚举成员的值：

```ts twoslash
// @errors: 2322
enum ShapeKind {
  Circle,
  Square,
}

interface Circle {
  kind: ShapeKind.Circle
  radius: number
}

interface Square {
  kind: ShapeKind.Square
  sideLength: number
}

let c: Circle = {
  kind: ShapeKind.Square,
  radius: 100,
}
```

另一个变化是枚举类型本身实际上变成了每个枚举成员的*联合*（union）。
通过联合枚举，类型系统能够利用它准确知晓枚举中存在哪些值这一特性。
正因如此，TypeScript 能够捕获到我们可能错误比较值的地方。
例如：

```ts twoslash
// @errors: 2367
enum E {
  Foo,
  Bar,
}

function f(x: E) {
  if (x !== E.Foo || x !== E.Bar) {
    //
  }
}
```

在该示例中，我们首先检查了 `x` 是否*不为* `E.Foo`。
如果该检查通过，那么 `||` 操作符就会发生短路，`if` 语句体就会执行。
然而，如果检查未通过，那么 `x` *只能*是 `E.Foo`，因此再去检查它是否*不等于* `E.Bar` 就没有任何意义了。

## 运行时中的枚举

枚举是在运行时真实存在的对象。
例如，以下枚举：

```ts twoslash
enum E {
  X,
  Y,
  Z,
}
```

实际上可以作为参数传递给函数：

```ts twoslash
enum E {
  X,
  Y,
  Z,
}

function f(obj: { X: number }) {
  return obj.X
}

// Works, since 'E' has a property named 'X' which is a number.
f(E)
```

## 编译时中的枚举

尽管枚举是在运行时真实存在的对象，但 `keyof` 关键字在枚举上的工作方式与对常规对象的预期有所不同。相反，你应该使用 `keyof typeof` 来获取一个将所有枚举键表示为字符串的类型。

```ts twoslash
enum LogLevel {
  ERROR,
  WARN,
  INFO,
  DEBUG,
}

/**
 * This is equivalent to:
 * type LogLevelStrings = 'ERROR' | 'WARN' | 'INFO' | 'DEBUG';
 */
type LogLevelStrings = keyof typeof LogLevel

function printImportant(key: LogLevelStrings, message: string) {
  const num = LogLevel[key]
  if (num <= LogLevel.WARN) {
    console.log('Log level key is:', key)
    console.log('Log level value is:', num)
    console.log('Log level message is:', message)
  }
}
printImportant('ERROR', 'This is a message')
```

### 反向映射（Reverse mappings）

除了为成员生成带有属性名的对象之外，数字枚举成员还会获得从枚举值到枚举名称的*反向映射*（reverse mapping）。
例如，在下面这个例子中：

```ts twoslash
enum Enum {
  A,
}

let a = Enum.A
let nameOfA = Enum[a] // "A"
```

TypeScript 会将其编译为如下 JavaScript 代码：

```ts twoslash
// @showEmit
enum Enum {
  A,
}

let a = Enum.A
let nameOfA = Enum[a] // "A"
```

在这段生成的代码中，枚举被编译成一个同时存储正向映射（`name` -> `value`）和反向映射（`value` -> `name`）的对象。
对其他枚举成员的引用始终会被生成为属性访问，绝不会被内联。

请牢记，字符串枚举成员*绝不会*生成反向映射。

### `const` 枚举

在大多数情况下，枚举都是非常理想的解决方案。
然而，有时要求会更为严苛。
为了避免在访问枚举值时承担额外生成代码以及额外间接访问的开销，可以使用 `const` 枚举。
常量枚举通过在枚举上使用 `const` 修饰符来定义：

```ts twoslash
const enum Enum {
  A = 1,
  B = A * 2,
}
```

`const` 枚举只能使用常量枚举表达式，并且与常规枚举不同，它们在编译期间会被完全移除。
`const` 枚举的成员会在使用处被内联。
这是可行的，因为 `const` 枚举不能包含计算成员。

```ts twoslash
const enum Direction {
  Up,
  Down,
  Left,
  Right,
}

let directions = [Direction.Up, Direction.Down, Direction.Left, Direction.Right]
```

在生成的代码中将变成：

```ts twoslash
// @showEmit
const enum Direction {
  Up,
  Down,
  Left,
  Right,
}

let directions = [Direction.Up, Direction.Down, Direction.Left, Direction.Right]
```

#### `const` 枚举的陷阱

内联枚举值起初看起来很直观，但却伴随着一些微妙的影响。
这些陷阱仅与*外部*（ambient）`const` 枚举（基本上就是 `.d.ts` 文件中的 `const` 枚举）以及在跨项目共享它们时有关；但如果你发布或使用 `.d.ts` 文件，这些陷阱很可能也会影响到你，因为 `tsc --declaration` 会将 `.ts` 文件转换为 `.d.ts` 文件。

1. 出于 [`isolatedModules` 文档](/tsconfig#references-to-const-enum-members) 中阐明的原因，该模式在根本上与外部 `const` 枚举不兼容。
   这意味着如果你发布了外部 `const` 枚举，下游使用者将无法在开启 [`isolatedModules`](/tsconfig#isolatedModules) 的同时使用这些枚举值。
2. 你很容易在编译时内联来自依赖项版本 A 的枚举值，而在运行时导入版本 B。
   如果不非常小心，版本 A 和版本 B 的枚举可能会有不同的值，从而导致[令人匪夷所思的 bug](https://github.com/microsoft/TypeScript/issues/5219#issue-110947903)，例如走入错误的 `if` 分支。
   这类 bug 尤为致命，因为通常自动化测试是在项目构建的同时使用相同的依赖版本运行的，从而会完全遗漏这类 bug。
3. [`importsNotUsedAsValues: "preserve"`](/tsconfig#importsNotUsedAsValues) 不会省略用作值的 `const` 枚举的导入，但外部 `const` 枚举无法保证运行时 `.js` 文件的存在。
   这些无法解析的导入会在运行时引发错误。
   而明确省略导入的常规方式——[仅类型导入（type-only imports）](/docs/handbook/modules/reference.html#type-only-imports-and-exports)，目前[不允许使用 const 枚举值](https://github.com/microsoft/TypeScript/issues/40344)。

以下是避免这些陷阱的两种方法：

1. 完全不使用 `const` 枚举。
   借助 linter，你可以轻松[禁用 const 枚举](https://typescript-eslint.io/linting/troubleshooting#how-can-i-ban-specific-language-feature)。
   显然这可以规避 `const` 枚举带来的任何问题，但会阻止你的项目内联自身的枚举。
   与内联其他项目的枚举不同，内联项目自身的枚举并不会带来问题，并且对性能有所帮助。
2. 不发布外部 `const` 枚举，借助 [`preserveConstEnums`](/tsconfig#preserveConstEnums) 将它们去常量化（deconstifying）。
   这也是 [TypeScript 项目本身](https://github.com/microsoft/TypeScript/pull/5422) 内部采用的做法。
   [`preserveConstEnums`](/tsconfig#preserveConstEnums) 会为 `const` 枚举生成与普通枚举相同的 JavaScript 代码。
   然后，你可以在[构建步骤中](https://github.com/microsoft/TypeScript/blob/1a981d1df1810c868a66b3828497f049a944951c/Gulpfile.js#L144)安全地从 `.d.ts` 文件中剥离 `const` 修饰符。

   这样一来，下游使用者就不会内联来自你项目的枚举，从而规避了上述陷阱；同时与彻底禁用 `const` 枚举不同，项目自身依然可以内联自己的枚举。

## 外部枚举（Ambient enums）

外部枚举用来描述已经存在的枚举类型的结构。

```ts twoslash
declare enum Enum {
  A = 1,
  B,
  C = 2,
}
```

外部枚举与非外部枚举之间的一个重要区别在于：在常规枚举中，如果前一个枚举成员被视为常量，那么没有初始化器的成员也将被视为常量。
相比之下，没有初始化器的外部（且非 const）枚举成员*始终*被视为计算成员。

## 对象与枚举（Objects vs Enums）

在现代 TypeScript 中，当使用带有 `as const` 的对象就能满足需求时，你可能并不需要枚举：

```ts twoslash
const enum EDirection {
  Up,
  Down,
  Left,
  Right,
}

const ODirection = {
  Up: 0,
  Down: 1,
  Left: 2,
  Right: 3,
} as const

EDirection.Up
//         ^?

ODirection.Up
//         ^?

// Using the enum as a parameter
function walk(dir: EDirection) {}

// It requires an extra line to pull out the values
type Direction = (typeof ODirection)[keyof typeof ODirection]
function run(dir: Direction) {}

walk(EDirection.Left)
run(ODirection.Right)
```

相比 TypeScript 的 `enum`，支持这种格式的最大理由在于它能使你的代码库与 JavaScript 的现有规范保持一致；而[when/if](https://github.com/rbuckton/proposal-enum)枚举被正式添加到 JavaScript 中时，你便可以直接迁移到该语法。
