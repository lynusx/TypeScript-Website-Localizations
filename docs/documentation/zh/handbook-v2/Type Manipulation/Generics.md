---
title: 泛型
layout: docs
permalink: /zh/docs/handbook/2/generics.html
oneline: 接收参数的类型
---

在软件工程中，构建不仅具有明确且一致的 API，同时还具备良好复用性的组件是至关重要的。
能够同时处理当前数据以及未来数据的组件，将为构建大型软件系统提供最灵活的能力。

在像 C# 和 Java 这样的语言中，创建可复用组件的核心工具之一就是*泛型*（generics）—— 即能够创建可以处理多种类型而非单一类型的组件。
这使得用户可以使用这些组件并传入他们自己的类型。

## 泛型的 Hello World

首先，让我们编写泛型的“hello world”示例：恒等函数（identity function）。
恒等函数是一个会原样返回传入内容的函数。
你可以把它理解为类似于 `echo` 命令的操作。

如果不使用泛型，我们必须为恒等函数指定一个具体类型：

```ts twoslash
function identity(arg: number): number {
  return arg
}
```

或者，我们可以使用 `any` 类型来描述恒等函数：

```ts twoslash
function identity(arg: any): any {
  return arg
}
```

虽然使用 `any` 确实达到了“泛型”的目的，因为它使得函数可以接收任何类型的 `arg`，但当函数返回时，我们实际上丢失了该类型的具体信息。
如果我们传入一个数字，我们所能获得的唯一信息就只是它可以返回任何类型。

相反，我们需要一种能够捕获参数类型的方法，以便将其用于表示返回值的类型。
在这里，我们将使用*类型变量*（type variable）—— 一种作用于类型而非值的特殊变量。

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg
}
```

我们现在为恒等函数添加了一个类型变量 `Type`。
这个 `Type` 允许我们捕获用户提供的类型（例如 `number`），以便我们稍后使用该信息。
在这里，我们再次将 `Type` 用作返回类型。经过检查，我们可以看到参数和返回类型使用了相同的类型。
这使我们能够将类型信息在函数的一端传入，并从另一端输出。

我们称这个版本的 `identity` 函数是泛型的，因为它可以适用于多种类型。
与使用 `any` 不同，它与最初将参数和返回值类型固定为数字的 `identity` 函数一样精确（即不会丢失任何信息）。

编写好泛型恒等函数后，我们可以通过两种方式调用它。
第一种方式是将所有参数（包括类型参数）都传递给函数：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg
}
// ---cut---
let output = identity<string>('myString')
//       ^?
```

这里我们显式地将 `Type` 指定为 `string`，作为函数调用参数的一部分，通过围绕参数的 `<>` 而非 `()` 来表示。

第二种方式也是最常见的方式。在这里我们使用*类型参数推断*（type argument inference）—— 即我们希望编译器根据我们传入的参数类型自动为我们确定 `Type` 的值：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg
}
// ---cut---
let output = identity('myString')
//       ^?
```

注意，我们不需要在尖括号（`<>`）中显式传入类型；编译器只通过检查值 `"myString"`，就将 `Type` 设置为了它的类型。
虽然类型参数推断是一个有助于保持代码简短且更具可读性的实用工具，但在编译器无法推断类型的更复杂情况下，你可能仍需要像前一个示例那样显式传入类型参数。

## 使用泛型类型变量

当开始使用泛型时，你会发现创建像 `identity` 这样的泛型函数时，编译器会强制要求在函数体中正确使用任何泛型类型参数。
也就是说，你必须切实将这些参数视为可能是任意类型来对待。

以我们之前的 `identity` 函数为例：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg
}
```

如果我们希望在每次调用时都将参数 `arg` 的长度打印到控制台呢？
我们可能会尝试这样写：

```ts twoslash
// @errors: 2339
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length)
  return arg
}
```

当我们这样做时，编译器会报错指出我们正在访问 `arg` 的 `.length` 成员，但我们在任何地方都没有说明 `arg` 拥有该成员。
请记住，我们前面提到这些类型变量代表任何及所有类型，因此调用此函数的人完全可能传入一个 `number`，而数字是没有 `.length` 成员的。

假设我们实际上希望此函数处理的是 `Type` 的数组，而不是直接处理 `Type`。既然是处理数组，那么 `.length` 成员就应当是可用的。
我们可以像创建其他类型的数组一样来描述它：

```ts twoslash {1}
function loggingIdentity<Type>(arg: Type[]): Type[] {
  console.log(arg.length)
  return arg
}
```

你可以将 `loggingIdentity` 的类型理解为：“泛型函数 `loggingIdentity` 接收一个类型参数 `Type` 和一个参数 `arg`（它是 `Type` 的数组），并返回一个 `Type` 的数组。”
如果我们传入一个数字数组，我们将返回一个数字数组，因为 `Type` 会绑定到 `number`。
这允许我们将泛型类型变量 `Type` 作为我们正在处理的类型的一部分，而不是整个类型，从而赋予我们更大的灵活性。

或者，我们也可以这样编写相同的示例：

```ts twoslash {1}
function loggingIdentity<Type>(arg: Array<Type>): Array<Type> {
  console.log(arg.length) // 数组具有 .length，因此不再报错
  return arg
}
```

你可能已经在其他语言中熟悉了这种风格的类型。
在下一节中，我们将介绍如何创建类似于 `Array<Type>` 的自定义泛型类型。

## 泛型类型

在前面的章节中，我们创建了适用于多种类型的泛型恒等函数。
在本节中，我们将探讨函数本身的类型以及如何创建泛型接口。

泛型函数的类型与非泛型函数非常类似，只是类型参数排在最前面，类似于函数声明：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg
}

let myIdentity: <Type>(arg: Type) => Type = identity
```

我们也可以在类型中使用不同的泛型类型参数名称，只要类型变量的数量以及它们的使用方式能够对应匹配即可：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg
}

let myIdentity: <Input>(arg: Input) => Input = identity
```

我们还可以将泛型类型写为对象字面量类型的调用签名：

```ts twoslash
function identity<Type>(arg: Type): Type {
  return arg
}

let myIdentity: { <Type>(arg: Type): Type } = identity
```

这引出了我们编写的第一个泛型接口。
让我们将前一个示例中的对象字面量提取为一个接口：

```ts twoslash
interface GenericIdentityFn {
  <Type>(arg: Type): Type
}

function identity<Type>(arg: Type): Type {
  return arg
}

let myIdentity: GenericIdentityFn = identity
```

在类似的示例中，我们可能希望将泛型参数提升为整个接口的参数。
这让我们可以清楚地看到接口针对哪些类型进行了泛型化（例如 `Dictionary<string>` 而不仅仅是 `Dictionary`）。
这使得类型参数对接口的所有其他成员都可见。

```ts twoslash
interface GenericIdentityFn<Type> {
  (arg: Type): Type
}

function identity<Type>(arg: Type): Type {
  return arg
}

let myIdentity: GenericIdentityFn<number> = identity
```

请注意，我们的示例发生了一些细微的变化。
我们不再是描述一个泛型函数，而是拥有了一个作为泛型类型一部分的非泛型函数签名。
当我们使用 `GenericIdentityFn` 时，现在还需要指定相应的类型参数（这里是 `number`），从而锁定了底层调用签名所使用的类型。
理解何时将类型参数直接放在调用签名上，何时放在接口本身上，将有助于清楚表达类型的哪些方面是泛型的。

除了泛型接口，我们还可以创建泛型类。
请注意，无法创建泛型枚举和泛型命名空间。

## 泛型类

泛型类的形式与泛型接口类似。
泛型类在类名后跟随尖括号（`<>`）包裹的泛型类型参数列表。

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

这是对 `GenericNumber` 类非常字面化的使用，但你可能已经注意到，没有任何限制要求它只能使用 `number` 类型。
我们完全可以使用 `string` 或更复杂的对象。

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

就像接口一样，将类型参数放在类本身上可以确保类的所有属性都使用相同的类型。

正如我们在[类相关章节](/docs/handbook/2/classes.html)中所介绍的，类的类型分为两部分：静态部分和实例部分。
泛型类只对其其实例部分而非静态部分具有泛型性，因此在使用类时，静态成员不能使用类的类型参数。

## 泛型约束

回顾前面的示例，有时你可能想要编写一个作用于一组类型的泛型函数，而你对这组类型所具备的能力已有*一定*了解。
在我们的 `loggingIdentity` 示例中，我们希望能够访问 `arg` 的 `.length` 属性，但编译器无法证明每种类型都具有 `.length` 属性，因此它警告我们不能做出这种假设。

```ts twoslash
// @errors: 2339
function loggingIdentity<Type>(arg: Type): Type {
  console.log(arg.length)
  return arg
}
```

我们不希望它作用于任意及所有类型，而是希望限制该函数只能处理*同时*具有 `.length` 属性的类型。
只要类型具有该成员，我们就允许使用它，但要求必须至少具有该成员。
为此，我们必须将此要求作为 `Type` 范围的约束条件列出。

为此，我们将创建一个描述该约束的接口。
在这里，我们创建一个包含单一 `.length` 属性的接口，然后使用该接口配合 `extends` 关键字来表示我们的约束：

```ts twoslash
interface Lengthwise {
  length: number
}

function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length) // 现在我们知道它具有 .length 属性，因此不再报错
  return arg
}
```

由于该泛型函数现在受到了约束，它将不再适用于任意所有类型：

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

相反，我们需要传入类型具备所有必需属性的值：

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

你可以声明一个被另一个类型参数约束的类型参数。
例如，在这里我们希望根据属性名获取对象上的某个属性。
我们希望确保不会意外获取到 `obj` 上不存在的属性，因此我们在两个类型之间放置一个约束：

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

在 TypeScript 中使用泛型创建工厂函数时，需要通过构造函数来引用类类型。例如：

```ts twoslash
function create<Type>(c: { new (): Type }): Type {
  return new c()
}
```

更高级的示例使用 prototype 属性来推断并约束构造函数与类类型的实例部分之间的关系。

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

这种模式被用来支持 [mixins](/docs/handbook/mixins.html) 设计模式。

## 泛型参数默认值

通过为泛型类型参数声明默认值，可以使指定相应的类型参数变为可选。例如，一个创建新 `HTMLElement` 的函数。在不传参调用时生成 `HTMLDivElement`；以某个元素作为第一个参数调用时生成与该参数类型相同的元素。你还可以选择传入子元素列表。以前你必须将该函数定义为：

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

使用泛型参数默认值后，我们可以将其简化为：

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

- 如果一个类型参数拥有默认值，则它被视为可选的。
- 必选类型参数不得出现在可选类型参数之后。
- 类型参数的默认类型必须满足该类型参数的约束（如果存在约束的话）。
- 在指定类型参数时，只需为必选类型参数指定类型实参。未指定的类型参数将解析为其默认类型。
- 如果指定了默认类型且推断无法得出候选类型，则推断为该默认类型。
- 与现有类或接口声明合并的类或接口声明可以为现有类型参数引入默认值。
- 与现有类或接口声明合并的类或接口声明可以引入新的类型参数，只要它指定了默认值即可。

## 型变注解

> 这是一项用于解决非常特定问题的高级功能，仅应在你明确找出使用它的理由时使用

[协变与逆变](<https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science)>)是类型理论中的术语，用于描述两个泛型类型之间的关系。
以下是关于该概念的简要入门介绍。

例如，如果你有一个表示可以 `make` 某种类型的对象的接口：

```ts
interface Producer<T> {
  make(): T
}
```

我们可以在期望使用 `Producer<Animal>` 的地方使用 `Producer<Cat>`，因为 `Cat` 是一种 `Animal`。
这种关系被称为*协变*（covariance）：从 `Producer<T>` 到 `Producer<U>` 的关系与从 `T` 到 `U` 的关系相同。

相反，如果你有一个可以 `consume` 某种类型的接口：

```ts
interface Consumer<T> {
  consume: (arg: T) => void
}
```

那么我们可以在期望使用 `Consumer<Cat>` 的地方使用 `Consumer<Animal>`，因为任何能够接收 `Animal` 的函数也必然能够接收 `Cat`。
这种关系被称为*逆变*（contravariance）：从 `Consumer<T>` 到 `Consumer<U>` 的关系与从 `U` 到 `T` 的关系相反。
请注意与协变相比方向发生了反转！这就是为什么逆变会“相互抵消”，而协变不会。

在像 TypeScript 这样的结构化类型系统中，协变和逆变是遵循类型定义自然涌现的行为。
即使在没有泛型的情况下，我们也能看到协变（和逆变）关系：

```ts
interface AnimalProducer {
  make(): Animal
}

// CatProducer 可以用于任何
// 期望 Animal 生产者的地方
interface CatProducer {
  make(): Cat
}
```

TypeScript 采用结构化类型系统，因此在比较两个类型时（例如查看是否可以在期望使用 `Producer<Animal>` 的地方使用 `Producer<Cat>`），通常的算法是结构化展开这两个定义，并比较它们的结构。
然而，型变（variance）带来了一项极其有用的优化：如果 `Producer<T>` 在 `T` 上是协变的，那么我们只需检查 `Cat` 和 `Animal` 即可，因为我们知道它们之间的关系与 `Producer<Cat>` 和 `Producer<Animal>` 相同。

请注意，只有在检查同一类型的两个具现化实例时才能使用此逻辑。
如果我们有 `Producer<T>` 和 `FastProducer<U>`，无法保证 `T` 和 `U` 一定指代这些类型中的相同位置，因此该检查将始终按结构化方式执行。

因为型变是结构化类型的自然属性，TypeScript 会自动*推断*每个泛型类型的型变。
**在极少数情况下**（涉及某些类型的循环引用类型），此计算可能会不准确。
如果发生这种情况，你可以为类型参数添加型变注解（variance annotation）以强制指定某种型变：

```ts
// 逆变标注
interface Consumer<in T> {
  consume: (arg: T) => void
}

// 协变标注
interface Producer<out T> {
  make(): T
}

// 不变标注
interface ProducerConsumer<in out T> {
  consume: (arg: T) => void
  make(): T
}
```

只有在你编写的型变与在结构上*应当*发生的型变一致时，才这样做。

> 绝不要编写与结构型变不匹配的型变注解！

必须强调的是，型变注解仅在基于实例化的比较（instantiation-based comparison）中生效。
它们在结构化比较中没有任何效果。
例如，你不能使用型变注解来“强制”使某个类型真正成为不变的（invariant）：

```ts
// 请勿这样做 —— 型变标注
// 与结构化行为不匹配
interface Producer<in out T> {
  make(): T
}

// 不是类型错误 —— 这是一个结构化
// 比较，因此型变标注不会生效
const p: Producer<string | number> = {
  make(): number {
    return 42
  },
}
```

在这里，对象字面量的 `make` 函数返回 `number`，我们可能期望这会导致错误，因为 `number` 并不是 `string | number`。
然而，这并不是基于实例化的比较，因为该对象字面量是一个匿名类型，而不是 `Producer<string | number>`。

> 型变注解不会改变结构化行为，并且只在特定情况下被参考

非常重要的一点是：只有当你完全清楚为什么这样做、它们的局限性是什么以及它们何时不生效时，才编写型变注解。
TypeScript 使用基于实例化的比较还是结构化比较并不是一项规范行为，出于正确性或性能方面的考量，不同版本之间可能会发生变化，因此你只应在型变注解与类型的结构化行为相匹配时才编写它。
不要试图使用型变注解来“强制”某种特定的型变；这会导致代码中出现不可预测的行为。

> 除非型变注解与类型的结构化行为相匹配，否则切勿编写型变注解

请记住，TypeScript 可以自动从你的泛型类型中推断型变。
几乎从不需要编写型变注解，只有在确认了具体需求时才应当这样做。
型变注解*不会*改变类型的结构化行为，并且根据具体情况，你可能会看到执行了结构化比较，而你原本期望的是基于实例化的比较。
型变注解不能用于修改类型在这些结构化上下文中的行为方式，并且除非该注解与结构化定义相同，否则不应编写。
因为这很难把握准确，而且 TypeScript 在绝大多数情况下都能正确推断型变，所以在常规代码中你不应该需要编写型变注解。

> 不要试图使用型变注解来改变类型检查行为；这并非它们的用途

在“类型调试”场景中，你*可能*会发现临时的型变注解很有用，因为型变注解会受到检查。
如果注解的型变明显有误，TypeScript 将会报错：

```ts
// 错误，此接口在 T 上绝对是逆变的
interface Foo<out T> {
  consume: (arg: T) => void
}
```

但是，型变注解允许比实际更严格（例如，如果实际型变是协变，则 `in out` 是合法的）。
调试完成后，请务必移除你的型变注解。

最后，如果你正在努力优化类型检查性能，*并且*已经运行了性能分析器，*并且*定位出了具体导致变慢的类型，*并且*确定了特别是型变推断变慢，*并且*仔细验证了你想要编写的型变注解，那么在极其复杂的类型中添加型变注解*可能*会带来微小的性能提升。

> 不要试图使用型变注解来改变类型检查行为；这并非它们的用途
