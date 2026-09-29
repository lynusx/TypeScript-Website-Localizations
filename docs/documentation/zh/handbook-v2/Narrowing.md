---
title: 收窄
layout: docs
permalink: /zh/docs/handbook/2/narrowing.html
oneline: '了解 TypeScript 如何利用 JavaScript 的特性来减少项目中的类型语法。'
---

假设我们有一个名为 `padLeft` 的函数。

```ts twoslash
function padLeft(padding: number | string, input: string): string {
  throw new Error('Not implemented yet!')
}
```

如果 `padding` 是 `number`，函数会将其视为要添加到 `input` 前的空格数。如果 `padding` 是 `string`，函数则直接将 `padding` 拼接在 `input` 前面。我们尝试实现 `padding` 传入 `number` 时的逻辑。

```ts twoslash
// @errors: 2345
function padLeft(padding: number | string, input: string): string {
  return ' '.repeat(padding) + input
}
```

糟糕，`padding` 报错了。TypeScript 警告我们，传递给 `repeat` 函数的值类型是 `number | string`，而该函数只接受 `number`，这个警告是对的。换句话说，我们没有事先显式检查 `padding` 是否为 `number`，也没有处理它是 `string` 的情况，那么我们就来把这些补上。

```ts twoslash
function padLeft(padding: number | string, input: string): string {
  if (typeof padding === 'number') {
    return ' '.repeat(padding) + input
  }
  return padding + input
}
```

如果这段代码看起来就是普通的 JavaScript 代码，那正是我们的目的所在。除了添加的类型注解，这段 TypeScript 代码与 JavaScript 别无二致。TypeScript 类型系统的理念是尽量让开发者轻松编写典型的 JavaScript 代码，而不必为了类型安全大费周章。

虽然看起来变化不大，但底层其实做了很多工作。就像 TypeScript 利用静态类型分析运行时值一样，它也将类型分析叠加在 JavaScript 的运行时控制流结构（如 `if/else`、三元条件、循环、真值检查等）之上，因为这些结构都会影响类型。

在 `if` 检查中，TypeScript 看到 `typeof padding === "number"`，并将其理解为一种特殊的代码形式 —— _类型守卫_。TypeScript 会追踪程序可能执行的路径，以分析给定位置上值最具体的类型。它会观察这些特殊的检查（即 _类型守卫_ ）和赋值，将类型细化为比声明时更具体的类型的过程，就称为 _收窄_。在许多编辑器中，我们可以观察到这些类型的变化，在接下来的示例中也会展示这一点。

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

TypeScript 能够理解多种用于收窄的结构。

## `typeof` 类型守卫

正如我们所见，JavaScript 支持 `typeof` 运算符，它能提供运行时值类型的基本信息。TypeScript 预期它会返回一组特定的字符串：

- `"string"`
- `"number"`
- `"bigint"`
- `"boolean"`
- `"symbol"`
- `"undefined"`
- `"object"`
- `"function"`

就像 `padLeft` 的例子一样，这个运算符在众多 JavaScript 库中频繁出现，TypeScript 能够理解它，从而在不同的分支中收窄类型。

在 TypeScript 中，检查 `typeof` 的返回值就是一种类型守卫。因为 TypeScript 编码了 `typeof` 对不同值的运算规则，所以它知道 JavaScript 中的一些怪异行为。例如，注意上面的列表，`typeof` 并不会返回字符串 `"null"`。看看下面这个例子：

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

在 `printAll` 函数中，我们试图通过检查 `strs` 是否为对象来判断它是否为数组类型（这里顺便复习一下，JavaScript 中的数组也是对象类型）。但事实证明，在 JavaScript 中，`typeof null` 的结果竟然是 `"object"`！这是历史遗留的意外之一。

经验丰富的用户可能不会感到惊讶，但并非所有人都遇到过这个 JavaScript 坑；幸运的是，TypeScript 会提示我们 `strs` 只被收窄为了 `string[] | null`，而不仅仅是 `string[]`。

这正好可以引出我们接下来要讨论的“真值”检查。

## 真值收窄

字典里可能查不到“Truthiness（真值）”这个词，但在 JavaScript 中你肯定会经常听到它。

在 JavaScript 中，我们可以在条件语句、`&&`、`||`、`if` 语句、布尔取反 (`!`) 等位置使用任何表达式。例如，`if` 语句并不要求其条件必须是 `boolean` 类型。

```ts twoslash
function getUsersOnlineMessage(numUsersOnline: number) {
  if (numUsersOnline) {
    return `There are ${numUsersOnline} online now!`
  }
  return "Nobody's here. :("
}
```

在 JavaScript 中，像 `if` 这样的结构首先会将条件“强制转换 (coerce)”为 `boolean` 类型以理解它们，然后根据结果是 `true` 还是 `false` 来选择执行分支。像这样的值：

- `0`
- `NaN`
- `""`（空字符串）
- `0n`（`bigint` 版本的零）
- `null`
- `undefined`

都会被转换为 `false`，而其他值则会被转换为 `true`。你总是可以通过 `Boolean` 函数或使用更简短的双重布尔取反，将值强制转换为 `boolean` 类型。（后者的优势在于 TypeScript 会将其推断为较窄的字面量布尔类型 `true`，而前者会被推断为 `boolean` 类型。）

```ts twoslash
// 以下两个结果均为 'true'
Boolean('hello') // type: boolean, value: true
!!'world' // type: true,    value: true
```

利用这种行为非常普遍，尤其是用来防范 `null` 或 `undefined` 这类值。下面我们尝试在 `printAll` 函数中应用它。

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

你会注意到，通过检查 `strs` 是否为真值，我们消除了上面的错误。这至少能防止在运行代码时出现类似下面这种可怕的错误：

```txt
TypeError: null is not iterable
```

但请记住，对基本类型进行真值检查往往容易出错。例如，看看另一种尝试编写 `printAll` 的方式：

```ts twoslash {class: "do-not-do-this"}
function printAll(strs: string | string[] | null) {
  // !!!!!!!!!!!!!!!!
  //  请勿这样做！
  //   继续往下看
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

我们把整个函数体包裹在真值检查中，但这带来了一个不易察觉的缺点：我们可能无法正确处理空字符串的情况了。

TypeScript 在这里并没有报错，但如果你对 JavaScript 不太熟悉，这种行为就值得注意。TypeScript 通常能帮你及早发现 bug，但如果你决定对某个值什么 _都不_ 做，它在不过度干预的前提下能做的也就只有这么多了。如果需要，你可以使用 linter 来确保处理此类情况。

关于真值收窄最后要说明的一点是，使用 `!` 进行布尔取反可以将值从被否定的分支中过滤出去。

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

TypeScript 还会使用 `switch` 语句以及像 `===`、`!==`、`==` 和 `!=` 这样的相等性检查来收窄类型。例如：

```ts twoslash
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // 我们现在可以在 'x' 或 'y' 上调用任何 'string' 方法。
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

在上面的例子中，当我们检查 `x` 和 `y` 是否相等时，TypeScript 知道它们的类型也必定相等。由于 `string` 是 `x` 和 `y` 共同拥有的唯一类型，TypeScript 就会知道在第一个分支中，`x` 和 `y` 都必须是 `string`。

与特定字面量值（而不是变量）进行等值检查同样有效。在关于真值收窄的小节中，我们编写了一个容易出错的 `printAll` 函数，因为它意外地没有正确处理空字符串。相反，我们可以通过特定检查来拦截 `null`，TypeScript 依然能正确地从 `strs` 的类型中移除 null。

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

JavaScript 中使用 `==` 和 `!=` 的宽松相等性检查也能被正确收窄。如果你不熟悉，检查某个值是否 `== null`，不仅会检查它是否具体为 `null`，还会检查它是否可能为 `undefined`。同理，`== undefined` 也会检查一个值是 `null` 还是 `undefined`。

```ts twoslash
interface Container {
  value: number | null | undefined
}

function multiplyValue(container: Container, factor: number) {
  // 从类型中移除 'null' 和 'undefined'。
  if (container.value != null) {
    console.log(container.value)
    //                    ^?

    // 现在我们可以安全地与 'container.value' 相乘。
    container.value *= factor
  }
}
```

## `in` 运算符收窄

JavaScript has an operator for determining if an object or its prototype chain has a property with a name: the `in` operator.
TypeScript takes this into account as a way to narrow down potential types.
JavaScript 提供了一个运算符，用于确定对象或其原型链是否包含特定名称的属性：即 `in` 运算符。TypeScript 会将其作为收窄潜在类型的一种方式。

例如，对于代码 `"value" in x`，其中 `"value"` 是字符串字面量，而 `x` 是联合类型。“true”分支会将 `x` 收窄为拥有可选或必选属性 `value` 的类型，而“false”分支则会收窄为拥有可选或缺失属性 `value` 的类型。

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

重申一下，对于收窄而言，可选属性在两个分支中都会存在。例如，人类既可以游泳也可以飞（借助合适的设备），因此在 `in` 检查的两个分支中都应该出现：

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

JavaScript 提供了一个运算符来检查某个值是否是另一个值的“实例”。更具体地说，在 JavaScript 中，`x instanceof Foo` 会检查 `x` 的 _原型链_ 中是否包含 `Foo.prototype`。虽然我们在这里不深入探讨（在介绍类的章节中会看到更多相关内容），但对于大多数可以用 `new` 构造的值来说，这仍然很有用。正如你可能猜到的那样，`instanceof` 也是一个类型守卫，TypeScript 会在使用 `instanceof` 守卫的分支中进行收窄。

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

## 赋值

正如前面提到的，当为变量赋值时，TypeScript 会观察赋值表达式的右侧，并相应地收窄左侧的类型。

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

注意，这些赋值都是合法的。尽管在第一次赋值后 `x` 的观察类型变为了 `number`，我们仍然能将 `string` 赋给 `x`。这是因为 `x` 的 _声明类型_（即它最初的类型）是 `string | number`，而赋值可行性始终是根据声明类型来检查的。

如果我们将 `boolean` 赋给 `x`，就会看到一个错误，因为这并不属于声明类型。

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

到目前为止，我们已经通过一些基本示例了解了 TypeScript 如何在特定分支内进行收窄。但这绝不仅仅是从每个变量出发、在 `if`、`while`、条件语句等结构中寻找类型守卫那么简单。例如：

```ts twoslash
function padLeft(padding: number | string, input: string) {
  if (typeof padding === 'number') {
    return ' '.repeat(padding) + input
  }
  return padding + input
}
```

`padLeft` 在其第一个 `if` 块中返回。TypeScript 能够分析这段代码，并发现在 `padding` 为 `number` 的情况下，剩余的函数体（`return padding + input;`）是 _不可达的_。因此，它能够将 `number` 从函数剩余部分中 `padding` 的类型里移除（从 `string | number` 收窄为 `string`）。

这种基于可达性的代码分析称为 _控制流分析_，TypeScript 会在遇到类型守卫和赋值时，利用这种流分析来收窄类型。在分析变量时，控制流可以不断地分岔和重新合并，并且我们可以观察到该变量在不同节点具有不同的类型。

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

到目前为止，我们都是利用现有的 JavaScript 结构来处理收窄；然而，有时你会希望更直接地控制类型在代码中的变化。

要自定义一个类型守卫，我们只需定义一个返回类型为 _类型谓词_ 的函数：

```ts twoslash
type Fish = { swim: () => void }
type Bird = { fly: () => void }
declare function getSmallPet(): Fish | Bird
// ---cut---
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined
}
```

在这个例子中，`pet is Fish` 就是我们的类型谓词。谓词的格式为 `parameterName is Type`，其中 `parameterName` 必须是当前函数签名中某个参数的名称。

只要用某个变量调用 `isFish`，如果其原始类型是兼容的，TypeScript 就会将该变量 _收窄_ 为那个特定的类型。

```ts twoslash
type Fish = { swim: () => void }
type Bird = { fly: () => void }
declare function getSmallPet(): Fish | Bird
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined
}
// ---cut---
// 对 'swim' 和 'fly' 的调用现在都没问题了。
let pet = getSmallPet()

if (isFish(pet)) {
  pet.swim()
} else {
  pet.fly()
}
```

注意，TypeScript 不仅知道 `pet` 在 `if` 分支中是 `Fish`；它还知道在 `else` 分支中你没有 `Fish`，所以必定是 `Bird`。

你可以使用类型守卫 `isFish` 来过滤 `Fish | Bird` 数组，从而获取一个 `Fish` 数组：

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
// 或者等价地
const underWater2: Fish[] = zoo.filter(isFish) as Fish[]

// 对于更复杂的示例，可能需要重复编写谓词
const underWater3: Fish[] = zoo.filter((pet): pet is Fish => {
  if (pet.name === 'sharkey') return false
  return isFish(pet)
})
```

此外，类可以 [使用 `this is Type`](/docs/handbook/2/classes.html#this-based-type-guards) 来收窄自身类型。

## 断言函数

类型也可以通过 [断言函数](/docs/handbook/release-notes/typescript-3-7.html#assertion-functions) 进行收窄。

# 可辨识联合

到目前为止，我们看过的大多数示例都集中于收窄包含 `string`、`boolean` 和 `number` 等简单类型的单一变量。虽然这很常见，但在 JavaScript 中，大多数情况下我们要处理的结构要稍复杂些。

为了提供些动力，设想我们正在尝试对圆形和正方形等形状进行编码。圆形需要记录半径，正方形需要记录边长。我们将使用一个名为 `kind` 的字段来表明正在处理的形状。下面是定义 `Shape` 的首次尝试。

```ts twoslash
interface Shape {
  kind: 'circle' | 'square'
  radius?: number
  sideLength?: number
}
```

注意，我们使用了一个字符串字面量类型的联合：`"circle"` 和 `"square"`，分别用来告诉我们该把形状当成圆形还是正方形处理。通过使用 `"circle" | "square"` 而非 `string`，我们可以避免拼写错误的问题。

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

我们可以编写一个 `getArea` 函数，根据处理的是圆还是正方形应用正确的逻辑。我们先试着处理圆形。

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

在开启 [`strictNullChecks`](/tsconfig#strictNullChecks) 时，这会抛出错误——这很合理，因为 `radius` 可能未定义。但是，如果我们在 `kind` 属性上执行相应的检查呢？

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

唔，TypeScript 依然不知道该怎么做。现在到了一个我们比类型检查器更了解值信息的阶段。我们可以尝试使用非空断言（在 `shape.radius` 后面加个 `!`）来表示 `radius` 绝对存在。

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

但这感觉并不理想。我们不得不对类型检查器使用那些非空断言 (`!`) 来让它相信 `shape.radius` 是有定义的，而且如果开始移动代码，这些断言就会很容易出错。此外，在关闭 [`strictNullChecks`](/tsconfig#strictNullChecks) 的情况下，无论如何我们都可能意外访问这些字段（因为读取可选属性时往往假定它们始终存在）。我们绝对能做得更好。

这种 `Shape` 编码的问题在于，类型检查器无法根据 `kind` 属性得知 `radius` 或 `sideLength` 是否存在。我们需要将 _我们_ 知道的信息传达给类型检查器。考虑到这一点，让我们重新定义 `Shape`。

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

在这里，我们适当地将 `Shape` 拆分成了两种类型，它们的 `kind` 属性具有不同的值，并且 `radius` 和 `sideLength` 分别在对应的类型中被声明为必填属性。

让我们看看现在尝试访问 `Shape` 的 `radius` 时会发生什么。

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

就像我们第一次定义 `Shape` 时一样，这仍然是个错误。当 `radius` 是可选的，我们会得到错误（在开启 [`strictNullChecks`](/tsconfig#strictNullChecks) 时），因为 TypeScript 无法判断该属性是否存在。而现在 `Shape` 是一个联合类型，TypeScript 提示我们 `shape` 可能是 `Square`，而 `Square` 并没有定义 `radius`！这两种解释都是对的，但无论 [`strictNullChecks`](/tsconfig#strictNullChecks) 如何配置，只有基于联合的 `Shape` 编码才会引发错误。

但是如果再次尝试检查 `kind` 属性呢？

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

错误消失了！当联合类型中的每个类型都包含带有字面量类型的公共属性时，TypeScript 会将其视为 _可辨识联合_，并能收窄联合的成员。

在此例中，`kind` 就是那个公共属性（被认为是 `Shape` 的 _判别 (discriminant)_ 属性）。检查 `kind` 属性是否为 `"circle"`，排除了 `Shape` 中所有 `kind` 类型不为 `"circle"` 的类型。这样就将 `shape` 收窄为 `Circle` 类型。

同样的检查对于 `switch` 语句也有效。现在我们可以尝试编写完整的 `getArea`，彻底摆脱那些讨厌的非空断言 `!`。

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

这里的关键在于 `Shape` 的编码方式。向 TypeScript 传达正确的信息 —— 即 `Circle` 和 `Square` 实际上是具有特定 `kind` 字段的两种独立类型 —— 至关重要。这样做让我们写出的类型安全的 TypeScript 代码，与普通 JavaScript 代码没有任何区别。得益于此，类型系统能够“正确”处理，并推断出 `switch` 语句各个分支中的类型。

> 顺便一提，试着把玩一下上面的示例，去掉几个 return 关键字。你会发现，在 `switch` 语句中意外穿透不同子句时，类型检查能帮你避免 bug。

可辨识联合的用处不仅限于圆和正方形。它们非常适合用来表示 JavaScript 中的任何消息传递机制，比如通过网络发送消息（客户端/服务器通信），或在状态管理框架中编码变动 (mutations)。

# `never` 类型

在收窄时，你可能会将联合类型的选项减少到排除所有可能性、什么也不剩的地步。在这些情况下，TypeScript 会使用 `never` 类型来表示这种不应存在的状态。

# 穷尽性检查

`never` 类型可以赋值给任何类型；然而，没有任何类型可以赋值给 `never`（除了 `never` 本身）。这意味着你可以使用收窄并依靠 `never` 类型在 `switch` 语句中进行穷尽性检查。

例如，在我们的 `getArea` 函数中添加一个 `default` 分支，尝试将形状赋值给 `never`，当所有可能的情况都已被处理完毕时，就不会引发错误。

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

向 `Shape` 联合添加一个新成员，将会导致 TypeScript 报错：

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
