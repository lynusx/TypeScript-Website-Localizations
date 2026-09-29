---
title: 泛型
layout: docs
permalink: /zh/docs/handbook/2/generics.html
oneline: '接受参数的类型'
---

软件工程的一个重要部分是构建这样的组件：不仅具有定义明确且一致的 API，而且还是可复用的。既能处理今天的数据、也能处理明天的数据的组件，将为你构建大型软件系统提供最灵活的能力。

在 C# 和 Java 这样的语言中，创建可复用组件的主要工具之一是 _泛型_，即能够创建一个可以处理多种类型而不是单一类型的组件。这让用户可以使用自己的类型来消费这些组件。

## 泛型的 Hello World

首先，让我们来写泛型的 "hello world"：恒等函数（identity function）。恒等函数是一个会原样返回传入内容的函数。你可以把它理解成类似 `echo` 命令的东西。

不使用泛型的话，我们要么得给恒等函数一个具体的类型：

```ts twoslash
function identity(arg: number): number {
  return arg
}
```

要么用 `any` 类型来描述恒等函数：

```ts twoslash
function identity(arg: any): any {
  return arg
}
```

虽然使用 `any` 确实算得上"泛化"，因为它会让函数接受任何类型的 `arg`，但实际上我们丢失了函数返回时该类型是什么的信息。如果我们传入一个数字，我们唯一知道的信息是可能返回任何类型。

相反，我们需要一种捕获实参类型的方式，让我们也能用它来表示返回的内容。这里我们将使用 _类型变量_，一种作用于类型而非值的特殊变量。

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg
}
```

现在我们给恒等函数添加了一个类型变量 `Type`。这个 `Type` 允许我们捕获用户提供的类型（例如 `number`），以便我们稍后使用该信息。在这里，我们再次把 `Type` 用作返回类型。观察一下就能发现，现在实参和返回类型使用的是同一个类型。这让我们可以把类型信息从函数的一端传进去，再从另一端带出来。

我们说这个版本的 `identity` 函数是泛型的，因为它可以处理一系列类型。与使用 `any` 不同，它和第一个使用 `number` 作为实参和返回类型的 `identity` 函数一样精确（即不会丢失任何信息）。

第一种方式是将所有参数（包括类型实参）都传给函数：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg
}
// ---cut---
let output = identity<string>('myString')
//       ^?
```

这里我们显式地把 `Type` 设为 `string`，作为函数调用的实参之一，用 `<>` 而不是 `()` 来表示。

第二种方式也许是最常见的。这里我们使用 _类型实参推断_ —— 即我们希望编译器根据我们传入的参数类型，自动为我们设置 `Type` 的值：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg
}
// ---cut---
let output = identity('myString')
//       ^?
```

注意，我们不必在尖括号（`<>`）中显式传入类型；编译器只是查看了值 `"myString"`，然后把 `Type` 设为它的类型。虽然类型实参推断是让代码更短、更可读的有用工具，但在更复杂的例子中，当编译器无法推断类型时，你可能需要像前面的例子那样显式传入类型实参。

## 使用泛型类型变量

当你开始使用泛型时，你会注意到：当你创建像 `identity` 这样的泛型函数时，编译器会强制你在函数体内正确地使用任何泛型类型的参数。也就是说，你必须真正把这些参数当作可以是任何类型来对待。

让我们回顾一下之前的 `identity` 函数：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg
}
```

如果我们还想在每次调用时把实参 `arg` 的长度打印到控制台呢？我们可能会忍不住这样写：

```ts twoslash
// @errors: 2339
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length)
  return arg
}
```

当我们这样做时，编译器会报错：指出我们正在使用 `arg` 的 `.length` 成员，但我们并没有在任何地方声明 `arg` 具有该成员。记住，我们之前说过，这些类型变量代表任何类型，所以使用这个函数的人完全可能传入一个没有 `.length` 成员的 `number`。

假设我们实际上打算让这个函数处理 `Type` 的数组，而不是直接处理 `Type`。既然我们处理的是数组，`.length` 成员就应该是可用的。我们可以像创建其他类型的数组一样来描述这一点：

```ts twoslash {1}
function loggingIdentity<Type>(arg: Type[]): Type[] {
  console.log(arg.length)
  return arg
}
```

你可以把 `loggingIdentity` 的类型读作："泛型函数 `loggingIdentity` 接受一个类型参数 `Type`，以及一个是 `Type` 数组的实参 `arg`，并返回一个 `Type` 数组。"如果我们传入一个数字数组，就会得到一个数字数组返回，因为 `Type` 会绑定到 `number`。这让我们可以把泛型类型变量 `Type` 作为我们要处理的类型的一部分，而不是整个类型，从而赋予我们极大的灵活性。

我们也可以把示例改写成这样：

```ts twoslash {1}
function loggingIdentity<Type>(arg: Array<Type>): Array<Type> {
  console.log(arg.length) // 数组拥有 .length 属性，所以不再报错
  return arg
}
```

你可能已经从其他语言熟悉了这种风格的类型。在下一节中，我们将介绍如何创建你自己的泛型类型，比如 `Array<Type>`。

## 泛型类型

在之前的章节中，我们创建了可以处理一系列类型的泛型恒等函数。在本节中，我们将探讨函数本身的类型，以及如何创建泛型接口。

泛型函数的类型与非泛型函数的类型类似，只是类型参数列在前面，类似于函数声明：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg
}

let myIdentity: <Type>(arg: Type) => Type = identity
```

我们也可以在类型中为泛型类型参数使用不同的名字，只要类型变量的数量和使用方式对得上即可。

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg
}

let myIdentity: <Input>(arg: Input) => Input = identity
```

我们还可以把泛型类型写成对象字面量类型的调用签名：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg
}

let myIdentity: { <Type>(arg: Type): Type } = identity
```

这就引出了我们的第一个泛型接口。让我们把上一个例子中的对象字面量移到接口中：

```ts twoslash
interface GenericIdentityFn {
  <Type>(arg: Type): Type
}

function identity<Type>(arg: Type): Type {
  return arg
}

let myIdentity: GenericIdentityFn = identity
```

在一个类似的例子中，我们可能想把泛型参数移到整个接口的参数上。这让我们能看到我们是针对什么类型泛化的（例如 `Dictionary<string>` 而不仅仅是 `Dictionary`）。这使得类型参数对接口的所有其他成员都可见。

```ts twoslash
interface GenericIdentityFn<Type> {
  (arg: Type): Type
}

function identity<Type>(arg: Type): Type {
  return arg
}

let myIdentity: GenericIdentityFn<number> = identity
```

注意，我们的例子已经变成了略有不同的东西。我们不再是描述一个泛型函数，而是拥有了一个作为泛型类型一部分的非泛型函数签名。当我们使用 `GenericIdentityFn` 时，现在还需要指定相应的类型实参（这里是 `number`），实际上锁定了底层调用签名将使用的内容。理解何时把类型参数直接放在调用签名上、何时放在接口本身上，将有助于描述类型的哪些方面是泛型的。

除了泛型接口，我们还可以创建泛型类。注意，无法创建泛型枚举和泛型命名空间。

## 泛型类

泛型类的结构与泛型接口相似。泛型类在类名后面的尖括号（`<>`）中有一个泛型类型参数列表。

```ts twoslash
// @strict: false
class GenericNumber<NumType> {
  zeroValue: NumType
  add: (x: NumType, y: NumType) => NumType
}

let myGenericNumber = new GenericNumber<number>()
myGenericNumber.zeroValue = 0
myGenericNumber.add = function (x, y) {
  return x + y
}
```

这是 `GenericNumber` 类的最直接用法，但你可能已经注意到，没有任何东西限制它只能使用 `number` 类型。我们完全可以改用 `string` 甚至更复杂的对象。

```ts twoslash
// @strict: false
class GenericNumber<NumType> {
  zeroValue: NumType
  add: (x: NumType, y: NumType) => NumType
}
// ---cut---
let stringNumeric = new GenericNumber<string>()
stringNumeric.zeroValue = ''
stringNumeric.add = function (x, y) {
  return x + y
}

console.log(stringNumeric.add(stringNumeric.zeroValue, 'test'))
```

与接口一样，把类型参数放在类本身上可以确保类的所有属性都使用相同的类型。

正如我们在 [类章节](/docs/handbook/2/classes.html) 中所介绍的，类的类型分为两面：静态部分和实例部分。泛型类仅对实例部分进行泛型化，而不是静态部分，因此在使用类时，静态成员不能使用类的类型参数。

## 泛型约束

如果你还记得前面的例子，有时你可能想写一个处理某组类型的泛型函数，而且你对这组类型将具有的能力有 _一定_ 的了解。在我们的 `loggingIdentity` 例子中，我们希望能够访问 `arg` 的 `.length` 属性，但编译器无法证明每个类型都有 `.length` 属性，所以它警告我们不能做这种假设。

```ts twoslash
// @errors: 2339
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length)
  return arg
}
```

我们不想处理任何类型，而是想把这个函数约束为处理任何 _同时_ 具有 `.length` 属性的类型。只要类型有这个成员，我们就允许它，但它必须至少具有这个成员。为此，我们必须把要求列为对 `Type` 可以是什么的约束。

为此，我们将创建一个描述约束的接口。在这里，我们创建一个只有一个 `.length` 属性的接口，然后使用这个接口和 `extends` 关键字来表示我们的约束：

```ts twoslash
interface Lengthwise {
  length: number
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length) // 现在我们知道它具备 .length 属性，所以不再报错
  return arg
}
```

因为泛型函数现在受到了约束，它不再能处理任意类型：

```ts twoslash
// @errors: 2345
interface Lengthwise {
  length: number
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length)
  return arg
}
// ---cut---
loggingIdentity(3)
```

相反，我们需要传入其类型具有所有必需属性的值：

```ts twoslash
interface Lengthwise {
  length: number
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length)
  return arg
}
// ---cut---
loggingIdentity({ length: 10, value: 3 })
```

## 在泛型约束中使用类型参数

你可以声明一个受另一个类型参数约束的类型参数。例如，在这里我们希望根据给定对象的属性名来获取对应的属性。我们希望确保不会意外获取 `obj` 上不存在的属性，因此我们在这两种类型之间放置了约束：

```ts twoslash
// @errors: 2345
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key]
}

let x = { a: 1, b: 2, c: 3, d: 4 }

getProperty(x, 'a')
getProperty(x, 'm')
```

## 在泛型中使用类类型

在 TypeScript 中使用泛型创建工厂时，必须通过其构造函数来引用类类型。例如：

```ts twoslash
function create<Type>(c: { new (): Type }): Type {
  return new c()
}
```

一个更高级的示例使用 prototype 属性来推断和约束构造函数与类类型的实例端之间的关系。

```ts twoslash
// @strict: false
class BeeKeeper {
  hasMask: boolean = true
}

class ZooKeeper {
  nametag: string = 'Mikle'
}

class Animal {
  numLegs: number = 4
}

class Bee extends Animal {
  numLegs = 6
  keeper: BeeKeeper = new BeeKeeper()
}

class Lion extends Animal {
  keeper: ZooKeeper = new ZooKeeper()
}

function createInstance<A extends Animal>(c: new () => A): A {
  return new c()
}

createInstance(Lion).keeper.nametag
createInstance(Bee).keeper.hasMask
```

这个模式被用于实现 [mixins](/docs/handbook/mixins.html) 设计模式。

## 泛型参数默认值

通过为泛型类型参数声明默认值，你可以让相应的类型实参变成可选的。例如，一个创建新 `HTMLElement` 的函数：不带实参调用该函数会生成一个 `HTMLDivElement`；以一个元素作为第一个实参调用该函数会生成实参类型的元素。你还可以选择传入一个子元素列表。以前你必须这样定义这个函数：

```ts twoslash
type Container<T, U> = {
  element: T
  children: U
}

// ---cut---
declare function create(): Container<HTMLDivElement, HTMLDivElement[]>
declare function create<T extends HTMLElement>(element: T): Container<T, T[]>
declare function create<T extends HTMLElement, U extends HTMLElement>(
  element: T,
  children: U[],
): Container<T, U[]>
```

使用泛型参数默认值，我们可以将其简化为：

```ts twoslash
type Container<T, U> = {
  element: T
  children: U
}

// ---cut---
declare function create<
  T extends HTMLElement = HTMLDivElement,
  U extends HTMLElement[] = T[],
>(element?: T, children?: U): Container<T, U>

const div = create()
//    ^?

const p = create(new HTMLParagraphElement())
//    ^?
```

泛型参数默认值遵循以下规则：

- 具有默认值的类型参数被视为可选的。
- 必需的类型参数不得跟在可选的类型参数之后。
- 类型参数的默认类型必须满足该类型参数的约束（如果存在约束的话）。
- 指定类型实参时，只需为必需的类型参数指定类型实参。未指定的类型参数将解析为其默认类型。
- 如果指定了默认类型且推断无法选择候选项，则推断为默认类型。
- 与现有类或接口声明合并的类或接口声明，可以为现有类型参数引入默认值。
- 与现有类或接口声明合并的类或接口声明，可以引入新的类型参数，只要它指定了默认值。

## 型变注解

> 这是一个旨在解决非常特定问题的高级功能，应仅在你明确知道需要使用它的场景下使用。

[协变与逆变](<https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)>)是类型论术语，用于描述两种泛型类型之间的关系。

例如，如果你有一个接口，表示可以 `make` 某种类型的对象：

```ts
interface Producer<T> {
  make(): T
}
```

我们可以在期望使用 `Producer<Animal>` 的地方使用 `Producer<Cat>`，因为 `Cat` 是一种 `Animal`。这种关系称为 _协变_：从 `Producer<T>` 到 `Producer<U>` 的关系与从 `T` 到 `U` 的关系相同。

反过来，如果你有一个可以 `consume` 特定类型的接口：

```ts
interface Consumer<T> {
  consume: (arg: T) => void
}
```

那么我们可以在期望使用 `Consumer<Cat>` 的地方使用 `Consumer<Animal>`，因为任何能够接受 `Animal` 的函数肯定也能接受 `Cat`。这种关系称为 _逆变_：`从 Consumer<T>` 到 `Consumer<U>` 的关系等同于从 `U` 到 `T` 的关系。注意，与协变相比，方向颠倒了！这就是为什么逆变会“抵消自身”，而协变不会。

在 TypeScript 这样的结构化类型系统中，协变和逆变是从类型定义自然涌现的行为。即使没有泛型，我们也会看到协变（和逆变）关系：

```ts
interface AnimalProducer {
  make(): Animal
}

// 可以在任何预期使用 Animal producer 的地方使用 CatProducer
interface CatProducer {
  make(): Cat
}
```

TypeScript 采用结构化类型系统，因此在比较两个类型时（例如，判断 `Producer<Cat>` 能否用在期望 `Producer<Animal>` 的地方），通常的算法是把这两个定义进行结构化展开，然后比较它们的结构。然而，型变实现了一种极其有用的优化：如果 `Producer<T>` 在 `T` 上是协变的，那么我们只需检查 `Cat` 和 `Animal`，因为我们知道它们之间的关系与 `Producer<Cat>` 和 `Producer<Animal>` 的关系相同。

注意，这种逻辑只能在我们检查同一类型的两个实例化时使用。如果我们有一个 `Producer<T>` 和一个 `FastProducer<U>`，无法保证 `T` 和 `U` 一定指向这些类型中的相同位置，因此这种检查总是会以结构化方式进行。

由于型变是结构化类型自然涌现的属性，TypeScript 会自动 _推断_ 每个泛型类型的型变。**在极为罕见的情况下**，涉及某些循环类型时，这种推断可能不准确。如果发生这种情况，你可以向类型参数添加型变注解，以强制指定特定的型变：

```ts
// 逆变注解
interface Consumer<in T> {
  consume: (arg: T) => void
}

// 协变注解
interface Producer<out T> {
  make(): T
}

// 不变注解
interface ProducerConsumer<in out T> {
  consume: (arg: T) => void
  make(): T
}
```

只有当你所写的型变与 _结构上本应_ 出现的型变相同时，才应该这样做。

> 永远不要写与结构型变不匹配的型变注解！

必须强调的是，型变注解仅在基于实例化的比较中生效。在结构比较中，它们没有任何作用。例如，你不能使用型变注解来“强制”某个类型实际上是不变的

```ts
// 别这样做 —— 型变注解与结构行为不匹配
interface Producer<in out T> {
  make(): T
}

// 不是类型错误 —— 这是结构化比较，所以型变注解没有生效
const p: Producer<string | number> = {
  make(): number {
    return 42
  },
}
```

在此处，对象字面量的 `make` 函数返回了 `number`，我们可能会期望这引起报错，因为 `number` 并不是 `string | number`。然而，这不是基于实例化的比较，因为对象字面量是匿名类型，而不是 `Producer<string | number>`。

> 型变注解不会改变结构上的行为，仅在特定场景中才会被参考。

非常重要的一点是：只有当你完全知道自己为什么这样做、了解其局限性以及它们何时不生效时，才应该编写变型注解。TypeScript 使用基于实例化的比较还是结构化比较，并不是规范化的行为，出于正确性或性能的原因，不同版本之间可能发生变化，因此你只应在型变注解与类型的结构行为一致时才编写它们。不要试图使用型变注解“强制”发生某种特定的型变；这会在你的代码中引起不可预测的行为。

> 除非型变注解与类型的结构行为一致，否则不要编写。

记住，TypeScript 可以自动从你的泛型类型中推断出型变。几乎没有必要编写型变注解，只有在确认了特定的需求时才应当这么做。型变注解 _不会_ 改变类型的结构行为，而且根据具体情况的不同，当你期望进行基于实例化的比较时，有可能会看到 TypeScript 执行的是结构化比较。型变注解无法用来修改类型在这些结构上下文中的行为，并且除非注解与结构定义一致，否则不应编写。因为这很难做对，而且 TypeScript 在绝大多数情况下都能正确推断变型，所以在正常代码中你不应该需要编写变型注解。

> 不要试图用型变注解来改变类型检查行为，这不是它们的用途。

在"类型调试"的情况下，你 _可能_ 会发现临时使用变型注解很有用，因为变型注解是会被检查的。如果注解的变型明显是错误的，TypeScript 会报错：

```ts
// 错误，该接口在 T 上绝对是逆变的
interface Foo<out T> {
  consume: (arg: T) => void
}
```

不过，允许使用比实际更严格的型变注解（例如，如果实际型变是协变的，使用 `in out` 是有效的）。调试完成后，请务必移除你的变型注解。

最后，如果你正在努力最大化类型检查性能，_并且_ 已经运行过性能分析器，_并且_ 已经确定了某个具体类型很慢，_并且_ 已经确定了是变型推断特别慢，_并且_ 已经仔细验证了你要写的变型注解，那么在极其复杂的类型上，你 _可能_ 通过添加变型注解获得一点性能收益。

> 不要试图用型变注解来改变类型检查行为；这不是它们的用途。
