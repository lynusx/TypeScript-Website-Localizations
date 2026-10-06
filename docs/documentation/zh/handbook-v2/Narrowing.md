---
title: 类型收窄
layout: docs
permalink: /zh/docs/handbook/2/narrowing.html
oneline: '了解 TypeScript 如何利用 JavaScript 运行时的认知来精简项目中的类型语法。'
---

试想我们有一个名为 `padLeft` 的函数。

```ts twoslash
function padLeft(padding: number | string, input: string): string {
  throw new Error('Not implemented yet!')
}
```

如果 `padding` 是 `number`，它会被视为我们希望在 `input` 前添加的空格数量。
如果 `padding` 是 `string`，它应该直接拼接到 `input` 的前面。
我们先来实现 `padLeft` 接收到 `number` 类型的 `padding` 时的逻辑。

```ts twoslash
// @errors: 2345
function padLeft(padding: number | string, input: string): string {
  return ' '.repeat(padding) + input
}
```

哎呀，`padding` 处报错了。
TypeScript 警告我们正在将一个类型为 `number | string` 的值传递给 `repeat` 函数，而该函数只接受 `number`，事实确实如此。
换句话说，我们既没有显式检查 `padding` 是否为 `number`，也没有处理它是 `string` 的情况。下面我们就来完善这些处理。

```ts twoslash
function padLeft(padding: number | string, input: string): string {
  if (typeof padding === 'number') {
    return ' '.repeat(padding) + input
  }
  return padding + input
}
```

如果这看起来大部分都是平淡无奇的 JavaScript 代码，这正是我们的设计初衷。
除了我们添加的类型注解之外，这段 TypeScript 代码看起来与普通 JavaScript 无异。
这是因为 TypeScript 的类型系统旨在尽可能简化常规 JavaScript 代码的编写，无需为了获得类型安全而刻意改变代码风格。

虽然表面上看起来平平无奇，但其底层其实大有乾坤。
正如 TypeScript 使用静态类型分析运行时值一样，它还将类型分析叠加在 JavaScript 的运行时控制流结构上，例如 `if/else`、三元条件运算符、循环以及真值检查等，这些结构都会对类型产生影响。

在 `if` 检查中，TypeScript 识别出 `typeof padding === "number"`，并将其理解为一种称为*类型保护*（_type guard_）的特殊代码形式。
TypeScript 会追踪程序可能执行的所有路径，进而分析出某个值在特定位置上所具有的最精确的类型。
它会分析这些特殊的检查（称为*类型保护*）以及赋值操作，将类型细化为比声明时更具体的类型的过程，就称为*类型收窄*（_narrowing_）。
在许多编辑器中，我们都可以实时观察到这些类型的变化，在接下来的示例中我们也会展示这一点。

```ts twoslash
function padLeft(padding: number | string, input: string): string {
  if (typeof padding === 'number') {
    return ' '.repeat(padding) + input
    //                ^?
  }
  return padding + input
  //     ^?
}
```

TypeScript 能够识别多种不同的代码结构来进行类型收窄。

## `typeof` 类型保护

正如我们所见，JavaScript 提供了 `typeof` 操作符，可以在运行时提供关于值的基本类型信息。
TypeScript 期望它返回以下固定的字符串集合：

- `"string"`
- `"number"`
- `"bigint"`
- `"boolean"`
- `"symbol"`
- `"undefined"`
- `"object"`
- `"function"`

正如在 `padLeft` 中看到的那样，这个操作符在许多 JavaScript 库中都非常常见，TypeScript 能够识别它以便在不同分支中收窄类型。

在 TypeScript 中，对 `typeof` 返回值进行检查就是一种类型保护。
因为 TypeScript 明确编码了 `typeof` 对不同值的运算规则，所以它也熟知 `typeof` 在 JavaScript 中的一些历史怪异行为。
例如，请注意在上面的列表中，`typeof` 并不会返回字符串 `"null"`。
请看以下示例：

```ts twoslash
// @errors: 2531 18047
function printAll(strs: string | string[] | null) {
  if (typeof strs === 'object') {
    for (const s of strs) {
      console.log(s)
    }
  } else if (typeof strs === 'string') {
    console.log(strs)
  } else {
    // do nothing
  }
}
```

在 `printAll` 函数中，我们尝试检查 `strs` 是否为对象，借此判断它是否为数组类型（这里正好需要强化一个认知：在 JavaScript 中数组也是对象类型）。
然而在 JavaScript 中，`typeof null` 实际返回的也是 `"object"`！
这是 JavaScript 历史遗留下来的一个遗憾设计。

经验丰富的开发者可能对此见怪不怪，但并不是每个人都在 JavaScript 中遇到过这个坑；幸运的是，TypeScript 会提醒我们 `strs` 仅被收窄到了 `string[] | null`，而不是我们期望的 `string[]`。

这正好引出了接下来的“真值”检查。

## 真值收窄

“真值”（Truthiness）可能不是词典里的正式词汇，但在 JavaScript 中却是一个随处可见的概念。

在 JavaScript 中，我们可以在条件语句、`&&`、`||`、`if` 语句、布尔非运算（`!`）等结构中使用任意表达式。
例如，`if` 语句并不要求其条件表达式的类型必须是 `boolean`。

```ts twoslash
function getUsersOnlineMessage(numUsersOnline: number) {
  if (numUsersOnline) {
    return `There are ${numUsersOnline} online now!`
  }
  return "Nobody's here. :("
}
```

在 JavaScript 中，诸如 `if` 之类的结构会先将其条件“强制转换”（coerce）为 `boolean` 才能理解其含义，然后根据转换结果是 `true` 还是 `false` 来选择执行分支。
以下值：

- `0`
- `NaN`
- `""`（空字符串）
- `0n`（`bigint` 版本的零）
- `null`
- `undefined`

都会被强制转换为 `false`，而其他值则会被转换为 `true`。
你随时可以通过 `Boolean` 函数将值转换为 `boolean`，或者使用更简洁的双重非运算（`!!`）。（后者的优势在于 TypeScript 会推断出更狭窄的字面量布尔类型 `true`，而前者推断出的类型则是 `boolean`。）

```ts twoslash
// both of these result in 'true'
Boolean('hello') // type: boolean, value: true
!!'world' // type: true,    value: true
```

利用这一特性非常普遍，特别是在防范 `null` 或 `undefined` 这类值的时候。
作为示例，让我们尝试在 `printAll` 函数中使用它。

```ts twoslash
function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === 'object') {
    for (const s of strs) {
      console.log(s)
    }
  } else if (typeof strs === 'string') {
    console.log(strs)
  }
}
```

你会发现，通过检查 `strs` 是否为真值，我们消除了上面的错误。
这至少能防止代码在运行时抛出令人头疼的错误，例如：

```txt
TypeError: null is not iterable
```

不过需要谨记的是，对原始值进行真值检查往往容易出错。
例如，考虑 `printAll` 的另一种写法：

```ts twoslash {class: "do-not-do-this"}
function printAll(strs: string | string[] | null) {
  // !!!!!!!!!!!!!!!!
  //  DON'T DO THIS!
  //   KEEP READING
  // !!!!!!!!!!!!!!!!
  if (strs) {
    if (typeof strs === 'object') {
      for (const s of strs) {
        console.log(s)
      }
    } else if (typeof strs === 'string') {
      console.log(strs)
    }
  }
}
```

我们将整个函数体包裹在一个真值检查中，但这存在一个不易察觉的缺陷：我们可能无法正确处理空字符串的情况。

TypeScript 在这里并不会提示错误，但如果你对 JavaScript 不够熟悉，这一行为非常值得留意。
TypeScript 通常可以帮助你在早期发现 bug，但如果你选择对某个值“什么都不做”，那么在不显得过于死板苛刻的前提下，类型系统所能做的也是有限的。
如果有需要，你可以借助 linter（代码检查工具）来确保此类情况得到妥善处理。

关于真值收窄的最后一点需要说明的是：通过 `!` 进行布尔取反可以在否定分支中过滤掉相应类型。

```ts twoslash
function multiplyAll(
  values: number[] | undefined,
  factor: number,
): number[] | undefined {
  if (!values) {
    return values
  } else {
    return values.map((x) => x * factor)
  }
}
```

## 等值收窄

TypeScript 还会使用 `switch` 语句以及像 `===`、`!==`、`==` 和 `!=` 这样的等值检查来收窄类型。
例如：

```ts twoslash
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // We can now call any 'string' method on 'x' or 'y'.
    x.toUpperCase()
    // ^?
    y.toLowerCase()
    // ^?
  } else {
    console.log(x)
    //          ^?
    console.log(y)
    //          ^?
  }
}
```

在上面的示例中，当我们检查 `x` 和 `y` 是否相等时，TypeScript 知道它们的类型也必然相等。
由于 `string` 是 `x` 和 `y` 都可以取到的唯一公共类型，因此 TypeScript 知道在第一个分支中 `x` 和 `y` 一定都是 `string`。

针对具体的字面量值（而非变量）进行检查同样有效。
在前面关于真值收窄的小节中，我们写过一个容易出错的 `printAll` 函数，因为它不小心把空字符串的情况给忽略掉了。
相反，我们可以进行针对性的检查来排除 `null`，TypeScript 依然能够正确地从 `strs` 的类型中移除 `null`。

```ts twoslash
function printAll(strs: string | string[] | null) {
  if (strs !== null) {
    if (typeof strs === 'object') {
      for (const s of strs) {
        //            ^?
        console.log(s)
      }
    } else if (typeof strs === 'string') {
      console.log(strs)
      //          ^?
    }
  }
}
```

JavaScript 中使用 `==` 和 `!=` 的宽松等值检查也能正确完成收窄。
如果你对此不太熟悉，检查某个值 `== null` 实际上不仅会检查它是否确为 `null`，还会检查它是否可能为 `undefined`。
同理，`== undefined` 也是如此：它会检查值是否为 `null` 或 `undefined` 中的任意一个。

```ts twoslash
interface Container {
  value: number | null | undefined
}

function multiplyValue(container: Container, factor: number) {
  // Remove both 'null' and 'undefined' from the type.
  if (container.value != null) {
    console.log(container.value)
    //                    ^?

    // Now we can safely multiply 'container.value'.
    container.value *= factor
  }
}
```

## `in` 操作符收窄

JavaScript 有一个用于判断对象或其原型链中是否存在指定名称属性的操作符：`in` 操作符。
TypeScript 将此作为一种收窄潜在类型的方式。

例如，对于代码 `"value" in x`，其中 `"value"` 是字符串字面量，而 `x` 是联合类型。
在“true”分支中，会将 `x` 收窄为具有名为 `value` 的可选属性或必需属性的类型；而在“false”分支中，则会收窄为具有可选 `value` 属性或完全不存在该属性的类型。

```ts twoslash
type Fish = { swim: () => void }
type Bird = { fly: () => void }

function move(animal: Fish | Bird) {
  if ('swim' in animal) {
    return animal.swim()
  }

  return animal.fly()
}
```

再次强调，可选属性在收窄时会同时存在于两侧的分支中。例如，人类既可以游泳也可以飞行（借助合适的装备），因此在 `in` 检查的两侧分支中都应该出现：

<!-- prettier-ignore -->
```ts twoslash
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = { swim?: () => void; fly?: () => void };

function move(animal: Fish | Bird | Human) {
  if ("swim" in animal) {
    animal;
//  ^?
  } else {
    animal;
//  ^?
  }
}
```

## `instanceof` 收窄

JavaScript 拥有一个用于检查某个值是否为另一个值的“实例”的操作符。
更具体地说，在 JavaScript 中，`x instanceof Foo` 用于检查 `x` 的*原型链*中是否包含 `Foo.prototype`。
虽然我们在这里不会深入展开（在我们介绍类的章节中会看到更多相关内容），但对于大多数可以使用 `new` 构造的值来说，它依然非常实用。
正如你可能猜到的那样，`instanceof` 也是一种类型保护，TypeScript 会在受 `instanceof` 保护的分支中收窄类型。

```ts twoslash
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString())
    //          ^?
  } else {
    console.log(x.toUpperCase())
    //          ^?
  }
}
```

## 赋值收窄

正如我们前面所提到的，当我们对任何变量进行赋值时，TypeScript 会查看赋值语句的右侧，并对左侧进行适当的收窄。

```ts twoslash
let x = Math.random() < 0.5 ? 10 : 'hello world!'
//  ^?
x = 1

console.log(x)
//          ^?
x = 'goodbye!'

console.log(x)
//          ^?
```

请注意，上述每个赋值都是合法的。
尽管在第一次赋值后观察到的 `x` 的类型变为了 `number`，但我们仍然能够将 `string` 赋值给 `x`。
这是因为 `x` 的*声明类型*（即 `x` 初始具有的类型）是 `string | number`，而可赋值性始终是根据声明类型来检查的。

如果我们尝试将 `boolean` 赋值给 `x`，就会看到一个错误，因为 `boolean` 并不属于声明类型的一部分。

```ts twoslash
// @errors: 2322
let x = Math.random() < 0.5 ? 10 : 'hello world!'
//  ^?
x = 1

console.log(x)
//          ^?
x = true

console.log(x)
//          ^?
```

## 控制流分析

到目前为止，我们已经了解了 TypeScript 在特定分支内收窄类型的一些基础示例。
但在实际运行中，事情远比单单从每个变量向上遍历并在 `if`、`while`、条件表达式中查找类型保护要复杂得多。
例如：

```ts twoslash
function padLeft(padding: number | string, input: string) {
  if (typeof padding === 'number') {
    return ' '.repeat(padding) + input
  }
  return padding + input
}
```

`padLeft` 在其第一个 `if` 代码块内部执行了 return 返回。
TypeScript 能够分析这段代码，并发现当 `padding` 为 `number` 时，函数体的其余部分（`return padding + input;`）是*不可达*（_unreachable_）的。
因此，对于函数的剩余部分，它能够将 `number` 从 `padding` 的类型中移除（将类型从 `string | number` 收窄为 `string`）。

这种基于可达性的代码分析称为*控制流分析*（_control flow analysis_），TypeScript 正是利用这种流分析在遇到类型保护和赋值语句时对类型进行收窄。
当对一个变量进行分析时，控制流可以不断分流和重新合并，并且可以观察到该变量在每个位置具有不同的类型。

```ts twoslash
function example() {
  let x: string | number | boolean

  x = Math.random() < 0.5

  console.log(x)
  //          ^?

  if (Math.random() < 0.5) {
    x = 'hello'
    console.log(x)
    //          ^?
  } else {
    x = 100
    console.log(x)
    //          ^?
  }

  return x
  //     ^?
}
```

## 使用类型谓词

到目前为止，我们一直利用现有的 JavaScript 结构来处理类型收窄，但有时你可能希望更直接地控制类型在代码中是如何变化的。

为了定义用户自定义的类型保护，我们只需要定义一个返回值类型为*类型谓词*（_type predicate_）的函数：

```ts twoslash
type Fish = { swim: () => void }
type Bird = { fly: () => void }
declare function getSmallPet(): Fish | Bird
// ---cut---
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined
}
```

在这个例子中，`pet is Fish` 就是我们的类型谓词。
谓词的形式为 `parameterName is Type`，其中 `parameterName` 必须是当前函数签名中的参数名称。

任何时候使用某个变量调用 `isFish` 时，如果该变量的原类型与目标类型兼容，TypeScript 就会将该变量*收窄*为那个特定类型。

```ts twoslash
type Fish = { swim: () => void }
type Bird = { fly: () => void }
declare function getSmallPet(): Fish | Bird
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined
}
// ---cut---
// Both calls to 'swim' and 'fly' are now okay.
let pet = getSmallPet()

if (isFish(pet)) {
  pet.swim()
} else {
  pet.fly()
}
```

请注意，TypeScript 不仅知道在 `if` 分支中 `pet` 是 `Fish`；
它还知道在 `else` 分支中，你拿到的*不是* `Fish`，因此必然是一个 `Bird`。

你可以使用类型保护 `isFish` 来过滤 `Fish | Bird` 类型的数组，从而获取 `Fish` 数组：

```ts twoslash
type Fish = { swim: () => void; name: string }
type Bird = { fly: () => void; name: string }
declare function getSmallPet(): Fish | Bird
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined
}
// ---cut---
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()]
const underWater1: Fish[] = zoo.filter(isFish)
// or, equivalently
const underWater2: Fish[] = zoo.filter(isFish) as Fish[]

// The predicate may need repeating for more complex examples
const underWater3: Fish[] = zoo.filter((pet): pet is Fish => {
  if (pet.name === 'sharkey') return false
  return isFish(pet)
})
```

此外，类还可以[使用 `this is Type`](/docs/handbook/2/classes.html#this-based-type-guards) 来收窄自身类型。

## 断言函数

类型还可以使用[断言函数](/docs/handbook/release-notes/typescript-3-7.html#assertion-functions)来进行收窄。

# 可辨识联合

到目前为止，我们看到的大多数示例都围绕着使用 `string`、`boolean` 和 `number` 等简单类型来收窄单个变量。
虽然这种情况很常见，但在 JavaScript 中大多数时候我们处理的都是稍微复杂一些的结构。

为了更直观地理解，设想我们正在尝试对圆形和正方形等形状进行建模。
圆形记录其半径，而正方形记录其边长。
我们将使用名为 `kind` 的字段来区分正在处理的是哪种形状。
这是定义 `Shape` 的第一次尝试：

```ts twoslash
interface Shape {
  kind: 'circle' | 'square'
  radius?: number
  sideLength?: number
}
```

请注意，我们使用了字符串字面量类型的联合：`"circle"` 和 `"square"`，分别用来指示我们应该将该形状视为圆形还是正方形。
通过使用 `"circle" | "square"` 而不是 `string`，我们可以避免拼写错误带来的问题。

```ts twoslash
// @errors: 2367
interface Shape {
  kind: 'circle' | 'square'
  radius?: number
  sideLength?: number
}

// ---cut---
function handleShape(shape: Shape) {
  // oops!
  if (shape.kind === 'rect') {
    // ...
  }
}
```

我们可以编写一个 `getArea` 函数，根据处理的是圆形还是正方形来应用对应的逻辑。
我们首先尝试处理圆形。

```ts twoslash
// @errors: 2532 18048
interface Shape {
  kind: 'circle' | 'square'
  radius?: number
  sideLength?: number
}

// ---cut---
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2
}
```

<!-- TODO -->

在开启 [`strictNullChecks`](/tsconfig#strictNullChecks) 的情况下，这会抛出错误——这是很合理的，因为 `radius` 可能未定义。
但如果我们对 `kind` 属性进行相应的检查呢？

```ts twoslash
// @errors: 2532 18048
interface Shape {
  kind: 'circle' | 'square'
  radius?: number
  sideLength?: number
}

// ---cut---
function getArea(shape: Shape) {
  if (shape.kind === 'circle') {
    return Math.PI * shape.radius ** 2
  }
}
```

唔，TypeScript 在这里仍然不知道该怎么做。
此时我们遇到了这样一种情况：我们对这些值的了解比类型检查器还要多。
我们可以尝试使用非空断言（在 `shape.radius` 后面加上 `!`）来表明 `radius` 绝对存在。

```ts twoslash
interface Shape {
  kind: 'circle' | 'square'
  radius?: number
  sideLength?: number
}

// ---cut---
function getArea(shape: Shape) {
  if (shape.kind === 'circle') {
    return Math.PI * shape.radius! ** 2
  }
}
```

但这感觉并不理想。
我们不得不通过这些非空断言（`!`）向类型检查器“大喊大叫”，以说服它 `shape.radius` 已定义，但如果我们开始调整代码结构，这些断言就很容易引发错误。
此外，在未开启 [`strictNullChecks`](/tsconfig#strictNullChecks) 的情况下，我们无论如何都可以意外访问这些字段（因为读取可选属性时，系统默认假定它们始终存在）。
我们完全可以做得更好。

这种定义 `Shape` 的方式的问题在于：类型检查器无法仅凭 `kind` 属性来推断 `radius` 或 `sideLength` 是否存在。
我们需要将*我们*所知道的信息传达给类型检查器。
考虑到这一点，让我们换个思路来定义 `Shape`。

```ts twoslash
interface Circle {
  kind: 'circle'
  radius: number
}

interface Square {
  kind: 'square'
  sideLength: number
}

type Shape = Circle | Square
```

在这里，我们将 `Shape` 正确拆分成了两个具有不同 `kind` 属性值的类型，并且 `radius` 和 `sideLength` 在它们各自的类型中都被声明为必需属性。

现在让我们看看当尝试访问 `Shape` 的 `radius` 时会发生什么。

```ts twoslash
// @errors: 2339
interface Circle {
  kind: 'circle'
  radius: number
}

interface Square {
  kind: 'square'
  sideLength: number
}

type Shape = Circle | Square

// ---cut---
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2
}
```

正如我们最初定义 `Shape` 时那样，这里依然会报错。
当 `radius` 是可选属性时，我们会得到一个错误（启用 [`strictNullChecks`](/tsconfig#strictNullChecks) 时），因为 TypeScript 无法判断该属性是否存在。
而现在 `Shape` 是一个联合类型，TypeScript 提示我们 `shape` 可能是 `Square`，而 `Square` 上根本没有定义 `radius`！
两种解释都是合理的，但只有联合类型的这种编码方式，无论 [`strictNullChecks`](/tsconfig#strictNullChecks) 如何配置都会产生错误。

但如果我们再次尝试检查 `kind` 属性呢？

```ts twoslash
interface Circle {
  kind: 'circle'
  radius: number
}

interface Square {
  kind: 'square'
  sideLength: number
}

type Shape = Circle | Square

// ---cut---
function getArea(shape: Shape) {
  if (shape.kind === 'circle') {
    return Math.PI * shape.radius ** 2
    //               ^?
  }
}
```

错误消失了！
当联合类型中的每个类型都包含一个具有字面量类型的公共属性时，TypeScript 会将其视为*可辨识联合*（_discriminated union_），并能够收窄该联合类型的成员。

在这种情况下，`kind` 就是那个公共属性（它被称为 `Shape` 的*可辨识属性*，即 _discriminant_ property）。
检查 `kind` 属性是否为 `"circle"` 会排除 `Shape` 中所有 `kind` 属性不为 `"circle"` 的类型。
这便将 `shape` 收窄为了 `Circle` 类型。

同样的检查方式也适用于 `switch` 语句。
现在我们可以尝试编写完整的 `getArea`，而无需任何恼人的 `!` 非空断言。

```ts twoslash
interface Circle {
  kind: 'circle'
  radius: number
}

interface Square {
  kind: 'square'
  sideLength: number
}

type Shape = Circle | Square

// ---cut---
function getArea(shape: Shape) {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2
    //                 ^?
    case 'square':
      return shape.sideLength ** 2
    //       ^?
  }
}
```

这里的关键在于 `Shape` 的类型设计方式。
向 TypeScript 传达正确的信息至关重要——即 `Circle` 和 `Square` 实际上是两个具有特定 `kind` 字段的独立类型。
这样一来，我们就能编写出既类型安全、又与平常手写的 JavaScript 毫无二致的 TypeScript 代码。
基于此，类型系统便能顺理成章地推断出 `switch` 语句中每个分支的正确类型。

> 顺便提一下，你可以尝试把玩上面的示例，删掉其中的某些 return 关键字。
> 你会发现，类型检查有助于避免因在 `switch` 语句中意外贯穿（fall through）不同分支而引发的 bug。

可辨识联合的用途远不止描述圆形和正方形。
它们非常适合在 JavaScript 中表示任何类型的消息传递方案，例如在网络上传输消息（客户端/服务端通信），或在状态管理框架中表示状态变更。

# `never` 类型

在进行类型收窄时，你可以将联合类型的可能选项缩减到排除所有可能性、什么都不剩的地步。
在这些情况下，TypeScript 会使用 `never` 类型来表示一个不应该存在的分支状态。

# 穷举检查

`never` 类型可以赋值给任何类型；但是，没有任何类型可以赋值给 `never`（除了 `never` 本身）。这意味着你可以利用类型收窄，并在 `switch` 语句中依赖出现的 `never` 来进行穷举检查。

例如，在我们的 `getArea` 函数中添加一个尝试将 shape 赋值给 `never` 的 `default` 分支，当所有可能的情况都已处理时，它不会产生任何错误。

```ts twoslash
interface Circle {
  kind: 'circle'
  radius: number
}

interface Square {
  kind: 'square'
  sideLength: number
}
// ---cut---
type Shape = Circle | Square

function getArea(shape: Shape) {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2
    case 'square':
      return shape.sideLength ** 2
    default:
      const _exhaustiveCheck: never = shape
      return _exhaustiveCheck
  }
}
```

向 `Shape` 联合类型中添加新成员时，就会引发 TypeScript 错误：

```ts twoslash
// @errors: 2322
interface Circle {
  kind: 'circle'
  radius: number
}

interface Square {
  kind: 'square'
  sideLength: number
}
// ---cut---
interface Triangle {
  kind: 'triangle'
  sideLength: number
}

type Shape = Circle | Square | Triangle

function getArea(shape: Shape) {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2
    case 'square':
      return shape.sideLength ** 2
    default:
      const _exhaustiveCheck: never = shape
      return _exhaustiveCheck
  }
}
```
