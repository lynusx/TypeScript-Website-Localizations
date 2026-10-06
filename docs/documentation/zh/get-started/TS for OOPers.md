---
title: 针对 Java/C# 开发者的 TypeScript 指南
short: 针对 Java/C# 开发者的 TS 指南
layout: docs
permalink: /zh/docs/handbook/typescript-in-5-minutes-oop.html
oneline: 针对具备面向对象语言背景的开发者学习 TypeScript
---

对于习惯了 C# 和 Java 等其他静态类型语言的程序员来说，TypeScript 是一个广受欢迎的选择。

TypeScript 的类型系统提供了许多相同的好处，例如更好的代码补全、更早的错误检测以及程序各组成部分之间更清晰的接口沟通。
尽管 TypeScript 为这类开发者提供了许多熟悉的特性，但我们仍然有必要退一步，看看 JavaScript（乃至 TypeScript）与传统的 OOP 语言之间有何不同。
理解这些差异有助于你编写出更出色的 JavaScript 代码，并避免从 C#/Java 直接转向 TypeScript 的程序员经常踩入的常见陷阱。

## 协同学习 JavaScript

如果你已经熟悉 JavaScript，但主要背景是 Java 或 C# 程序员，本入门指南可以帮助解释你可能会陷入的一些常见误区与陷阱。
TypeScript 对类型进行建模的某些方式与 Java 或 C# 截然不同，在学习 TypeScript 时牢记这些差异至关重要。

如果你是一名对 JavaScript 普遍缺乏了解的 Java 或 C# 程序员，我们建议先学习一些**不带**类型的 JavaScript，以了解 JavaScript 的运行时行为。
因为 TypeScript 不会改变代码的**运行**方式，所以你仍然需要了解 JavaScript 的运作方式，才能编写出切实执行任务的代码！

切记，TypeScript 与 JavaScript 使用相同的**运行时**，因此任何关于如何实现特定运行时行为的资源（如将字符串转换为数字、显示弹窗警告、将文件写入磁盘等），都同样适用于 TypeScript 程序。
不要将自己局限在仅针对 TypeScript 的资源中！

## 重新审视“类”

C# 和 Java 可以被称为**强制面向对象（mandatory OOP）**语言。
在这些语言中，**类（class）**是组织代码的基本单元，同时也是运行时容纳所有数据**与**行为的基本容器。
对于某些问题，强制将所有功能和数据封装在类中可能是一个不错的领域模型，但并非每个领域都**需要**以此方式来表达。

### 自由函数与数据

在 JavaScript 中，函数可以存在于任何地方，数据也可以自由传递，而无需置于预先定义的 `class` 或 `struct` 内部。
这种灵活性非常强大。
处理数据时使用不与类关联的“自由（free）”函数，且不引入隐含的 OOP 层级结构，往往是编写 JavaScript 程序的首选模式。

### 静态类

此外，C# 和 Java 中的某些结构（例如单例和静态类）在 TypeScript 中是没有必要的。

## TypeScript 中的 OOP

话虽如此，如果你愿意，依然可以使用类！
某些问题非常适合通过传统的 OOP 层级结构来解决，而 TypeScript 对 JavaScript 类的支持会让这些模型更加强大。
TypeScript 支持许多常见的设计模式，例如实现接口、继承以及静态方法。

我们将在本指南的后续部分介绍类。

## 重新审视“类型”

TypeScript 对**类型（type）**的理解实际上与 C# 或 Java 大不相同。
让我们一起来探讨其中的一些差异。

### 名义化具化类型系统

在 C# 或 Java 中，任何给定的值或对象都具有一个确切的类型——要么是 `null`、原始类型，要么是已知的类类型。
我们可以在运行时调用诸如 `value.GetType()` 或 `value.getClass()` 之类的方法来查询确切的类型。
该类型的定义会存在于某个具有特定名称的类中，并且除非存在显式的继承关系或共同实现的接口，否则我们不能将两个形状相似的类互相替代使用。

这些特征构成了一个**具化、名义（reified, nominal）**类型系统。
我们在代码中编写的类型在运行时是真实存在的，并且类型之间的关系是通过其声明来确立的，而非通过其结构。

### 类型即集合

在 C# 或 Java 中，将运行时类型与其编译时声明视为一一对应的关系是有意义的。

而在 TypeScript 中，最好将类型视为共享某些特性的**值的集合**。
因为类型只是集合，所以一个特定的值可以同时属于**多个**集合。

一旦你开始将类型视为集合，某些操作就会变得非常自然。
例如在 C# 中，传递一个**既可能是** `string` **又可能是** `int` 的值会很棘手，因为没有单个类型能够表示这种值。

在 TypeScript 中，一旦你意识到每个类型都只是一个集合，这一切就会变得非常自然。
你该如何描述一个要么属于 `string` 集合、要么属于 `number` 集合的值呢？
它仅仅属于这些集合的**并集（联合类型）**：`string | number`。

TypeScript 提供了许多基于集合论的方式来处理类型；如果你从集合的角度来思考类型，就会发现它们更加直观易懂。

### 擦除的结构化类型

在 TypeScript 中，对象**并不**属于单一确切的类型。
例如，如果我们构造了一个满足某个接口的对象，即便这两者之间没有声明式的继承或实现关系，我们也可以在需要该接口的地方直接使用该对象。

```ts twoslash
interface Pointlike {
  x: number
  y: number
}
interface Named {
  name: string
}

function logPoint(point: Pointlike) {
  console.log('x = ' + point.x + ', y = ' + point.y)
}

function logName(x: Named) {
  console.log('Hello, ' + x.name)
}

const obj = {
  x: 0,
  y: 0,
  name: 'Origin',
}

logPoint(obj)
logName(obj)
```

TypeScript 的类型系统是**结构化的（structural）**，而非名义上的（nominal）：我们可以将 `obj` 当作 `Pointlike` 来使用，因为它的 `x` 和 `y` 属性都是数字。
类型之间的关系取决于它们所包含的属性，而不是它们在声明时是否建立了某种特定关系。

TypeScript 的类型系统也**不是具化的（not reified）**：在运行时没有任何东西能告诉我们 `obj` 是 `Pointlike`。
事实上，`Pointlike` 类型在运行时**不以任何形式**存在。

回到**类型即集合**的观点，我们可以认为 `obj` 既是 `Pointlike` 值集合的成员，也是 `Named` 值集合的成员。

### 结构化类型的特性与影响

面向对象程序员经常对结构化类型系统的两个特定方面感到惊讶。

#### 空类型

第一个是**空类型（empty type）**的表现似乎出乎意料：

```ts twoslash
class Empty {}

function fn(arg: Empty) {
  // 执行某些操作？
}

// 没有报错，但这并不是一个 'Empty'？
fn({ k: 10 })
```

TypeScript 通过检查传入的实参是否是合法的 `Empty` 来判断这里对 `fn` 的调用是否有效。
它是通过检查 `{ k: 10 }` 与 `class Empty { }` 的**结构**来进行判断的。
我们可以看到 `{ k: 10 }` 具备 `Empty` 所拥有的**所有**属性，因为 `Empty` 本身没有任何属性。
因此，这是一个合法的调用！

这可能令人感到意外，但归根结底，它与名义化 OOP 语言所强制约束的关系非常相似。
子类不能**移除**基类的属性，因为这样做会破坏派生类与其基类之间天然的子类型关系。
结构化类型系统只是通过“具备兼容类型的属性”来隐式定义子类型，以此表达这种关系。

#### 相同类型

另一个经常令人感到意外的情况是结构相同的类型：

```ts
class Car {
  drive() {
    // 踩油门
  }
}
class Golfer {
  drive() {
    // 把球打远
  }
}

// 没有报错？
let w: Car = new Golfer()
```

同样，这里没有报错是因为这两个类的**结构**是完全相同的。
虽然这看似可能会引起困惑，但在实际开发中，结构完全相同却不应产生关联的类并不常见。

我们将在“类”章节中深入学习类与类之间的相互关系。

### 反射

面向对象程序员习惯于能够查询任何值的类型，即使是泛型类型：

```csharp
// C#
static void LogType<T>() {
    Console.WriteLine(typeof(T).Name);
}
```

由于 TypeScript 的类型系统会被完全擦除，因此在运行时无法获取诸如泛型类型参数的实例化信息。

JavaScript 确实拥有诸如 `typeof` 和 `instanceof` 等有限的原始操作符，但请记住，这些操作符处理的仍然是存在于类型擦除后的输出代码中的值。
例如，`typeof (new Car())` 的结果将是 `"object"`，而不是 `Car` 或 `"Car"`。

## 下一步

以上是对日常 TypeScript 中使用的语法和工具的简要概述。接下来，你可以：

- [从头到尾](/docs/handbook/intro.html)阅读完整手册
- 探索 [Playground 示例](/play#show-examples)
