---
title: 类型兼容性
layout: docs
permalink: /zh/docs/handbook/type-compatibility.html
oneline: TypeScript 中类型检查的工作原理
translatable: true
---

TypeScript 中的类型兼容性是基于结构子类型（structural subtyping）的。
结构类型是一种仅根据成员来关联类型的方式。
这与名义类型（nominal typing）形成对比。
请看以下代码：

```ts
interface Pet {
  name: string
}

class Dog {
  name: string
}

let pet: Pet
// 正常，因为结构化类型系统
pet = new Dog()
```

在诸如 C# 或 Java 这类名义类型的语言中，等价的代码将会报错，因为 `Dog` 类并未显式声明自己实现了 `Pet` 接口。

TypeScript 的结构类型系统是根据 JavaScript 代码的典型编写方式来设计的。
因为 JavaScript 中广泛使用函数表达式和对象字面量等匿名对象，所以相较于名义类型系统，使用结构类型系统能够更自然地表达 JavaScript 库中常见的各种关系。

## 关于健全性（Soundness）的说明

TypeScript 的类型系统允许某些在编译期无法确定是否安全的操作。当一个类型系统具备这种特性时，即被称作“不健全”（not sound）。TypeScript 允许这种不健全行为的地方都是经过深思熟虑的，在本文档中，我们将解释这些情况发生的位置以及其背后的设计考量。

## 开始入门

TypeScript 结构类型系统的基本规则是：如果 `y` 至少具有与 `x` 相同的成员，则 `x` 与 `y` 兼容。例如，考虑以下包含一个拥有 `name` 属性的 `Pet` 接口的代码：

```ts
interface Pet {
  name: string
}

let pet: Pet
// dog 推断出的类型为 { name: string; owner: string; }
let dog = { name: 'Lassie', owner: 'Rudd Weatherwax' }
pet = dog
```

要检查 `dog` 是否可以赋值给 `pet`，编译器会检查 `pet` 的每个属性，并在 `dog` 中寻找对应的兼容属性。
在此例中，`dog` 必须包含一个名为 `name` 的 string 类型成员。由于确实存在，因此赋值是合法的。

在检查函数调用参数时，也使用相同的赋值规则：

```ts
interface Pet {
  name: string
}

let dog = { name: 'Lassie', owner: 'Rudd Weatherwax' }

function greet(pet: Pet) {
  console.log('Hello, ' + pet.name)
}
greet(dog) // 正常
```

请注意，`dog` 包含一个额外的 `owner` 属性，但这并不会引发错误。
在检查兼容性时，只会考虑目标类型（本例中为 `Pet`）的成员。
这一比较过程是递归进行的，会深入检查每个成员和子成员的类型。

不过需要注意，对象字面量[只能指定已知属性](/docs/handbook/2/objects.html#excess-property-checks)。
例如，因为我们显式指定了 `dog` 的类型为 `Pet`，所以以下代码是无效的：

```ts
let dog: Pet = { name: 'Lassie', owner: 'Rudd Weatherwax' } // 错误
```

## 比较两个函数

虽然比较原始类型和对象类型相对直观，但判断哪些函数应被视为兼容则要复杂一些。
让我们先从两个仅在参数列表上存在差异的函数的基本示例开始：

```ts
let x = (a: number) => 0
let y = (b: number, s: string) => 0

y = x // 正常
x = y // 错误
```

要检查 `x` 是否可以赋值给 `y`，我们首先查看参数列表。
`x` 中的每个参数都必须在 `y` 中有一个具有兼容类型的对应参数。
注意，这里不考虑参数的名称，只考虑它们的类型。
在本例中，`x` 的每个参数都在 `y` 中找到了对应的兼容参数，因此允许赋值。

第二个赋值会报错，因为 `y` 拥有一个 `x` 所没有的必选第二个参数，所以不允许该赋值。

你可能会好奇为什么允许像示例中 `y = x` 那样“丢弃”参数。
允许这种赋值的原因在于，忽略多余的函数参数在 JavaScript 中实际上非常普遍。
例如，`Array#forEach` 向回调函数提供三个参数：数组元素、其索引以及数组本身。
尽管如此，提供一个仅使用第一个参数的回调函数是非常实用的：

```ts
let items = [1, 2, 3]

// 不要强行要求这些额外的参数
items.forEach((item, index, array) => console.log(item))

// 应该正常！
items.forEach((item) => console.log(item))
```

现在让我们来看看返回类型是如何处理的，以下示例中的两个函数仅在返回类型上有所不同：

```ts
let x = () => ({ name: 'Alice' })
let y = () => ({ name: 'Alice', location: 'Seattle' })

x = y // 正常
y = x // 错误，因为 x() 缺少 location 属性
```

类型系统要求源函数的返回类型必须是目标类型返回类型的子类型。

### 函数参数双向协变（Bivariance）

在比较函数参数的类型时，如果源参数可以赋值给目标参数，或者目标参数可以赋值给源参数，则赋值成功。
这是不健全的，因为调用者最终可能获得一个接受更具体类型的函数，但却用一个较宽泛的类型去调用它。
在实践中，这种错误很少发生，而允许这样做可以支持许多常见的 JavaScript 模式。简要示例如下：

```ts
enum EventType {
  Mouse,
  Keyboard,
}

interface Event {
  timestamp: number
}
interface MyMouseEvent extends Event {
  x: number
  y: number
}
interface MyKeyEvent extends Event {
  keyCode: number
}

function listenEvent(eventType: EventType, handler: (n: Event) => void) {
  /* ... */
}

// 不健全，但实用且常见
listenEvent(EventType.Mouse, (e: MyMouseEvent) => console.log(e.x + ',' + e.y))

// 追求健全性时的不理想替代方案
listenEvent(EventType.Mouse, (e: Event) =>
  console.log((e as MyMouseEvent).x + ',' + (e as MyMouseEvent).y),
)
listenEvent(EventType.Mouse, ((e: MyMouseEvent) =>
  console.log(e.x + ',' + e.y)) as (e: Event) => void)

// 仍然不允许（明确报错）。对于完全不兼容的类型强制保证类型安全
listenEvent(EventType.Mouse, (e: number) => console.log(e))
```

你可以通过编译器标志 [`strictFunctionTypes`](/tsconfig#strictFunctionTypes)，让 TypeScript 在出现这种情况时报错。

### 可选参数与剩余参数

在比较函数的兼容性时，可选参数和必选参数是可以互换的。
源类型中多余的可选参数不会引发错误，目标类型中没有对应源类型参数的可选参数也不会引发错误。

当函数拥有剩余参数（rest parameter）时，它会被视为一连串无限个可选参数。

从类型系统的角度来看，这并不健全；但从运行时角度来看，可选参数的概念通常并没有被严格强制执行，因为对于大多数函数而言，在相应位置传递 `undefined` 是等价的。

一个典型的用例模式是：一个函数接受一个回调，并使用对于程序员可预测、但对类型系统而言不可知数量的参数来调用它：

```ts
function invokeLater(args: any[], callback: (...args: any[]) => void) {
  /* ... 使用 'args' 调用回调 ... */
}

// 不健全 - invokeLater “可能”会提供任意数量的参数
invokeLater([1, 2], (x, y) => console.log(x + ', ' + y))

// 令人困惑（实际上需要 x 和 y）且难以发现
invokeLater([1, 2], (x?, y?) => console.log(x + ', ' + y))
```

### 带有重载的函数

当函数具有重载时，目标类型中的每个重载都必须与源类型上的一个兼容签名相匹配。
这确保了源函数能够在与目标函数完全相同的所有情况下被调用。

## 枚举

枚举与数字相互兼容，并且数字也与枚举相互兼容。来自不同枚举类型的枚举值被视为彼此不兼容。例如：

```ts
enum Status {
  Ready,
  Waiting,
}
enum Color {
  Red,
  Blue,
  Green,
}

let status = Status.Ready
status = Color.Green // 错误
```

## 类

类的工作方式与对象字面量类型和接口类似，但有一个例外：类同时拥有静态类型和实例类型。
在比较两个类类型的对象时，只会比较其实例成员。
静态成员和构造函数不会影响兼容性。

```ts
class Animal {
  feet: number
  constructor(name: string, numFeet: number) {}
}

class Size {
  feet: number
  constructor(numFeet: number) {}
}

let a: Animal
let s: Size

a = s // 正常
s = a // 正常
```

### 类中的私有成员和受保护成员

类中的私有成员和受保护成员会影响其兼容性。
当检查类的实例兼容性时，如果目标类型包含私有成员，则源类型也必须包含一个源自同一个类的私有成员。
同样，这也适用于具有受保护成员的实例。
这使得类与其父类在赋值上相互兼容，但与来自不同继承体系且碰巧具有相同形状的类之间**不**兼容。

## 泛型

由于 TypeScript 采用结构类型系统，类型参数只有在作为成员类型的一部分被使用时，才会影响最终生成的类型。例如：

```ts
interface Empty<T> {}
let x: Empty<number>
let y: Empty<string>

x = y // 正常，因为 y 匹配 x 的结构
```

在上面的代码中，`x` 和 `y` 是兼容的，因为它们的结构并没有以产生差异的方式使用类型实参。
给 `Empty<T>` 添加一个成员后，就可以看出它的运作方式：

```ts
interface NotEmpty<T> {
  data: T
}
let x: NotEmpty<number>
let y: NotEmpty<string>

x = y // 错误，因为 x 和 y 不兼容
```

通过这种方式，指定了类型实参的泛型类型的行为与非泛型类型完全相同。

对于未指定类型实参的泛型类型，在检查兼容性时会将所有未指定的类型实参替换为 `any`。
然后就像非泛型情况一样检查生成的类型的兼容性。

例如：

```ts
let identity = function <T>(x: T): T {
  // ...
}

let reverse = function <U>(y: U): U {
  // ...
}

identity = reverse // 正常，因为 (x: any) => any 匹配 (y: any) => any
```

## 高级主题

### 子类型 vs 赋值

到目前为止，我们一直在使用“兼容（compatible）”一词，但这并不是语言规范中定义的术语。
在 TypeScript 中，存在两种兼容性：子类型兼容性（subtype compatibility）与赋值兼容性（assignment compatibility）。
它们之间的区别仅在于：赋值兼容性在子类型兼容性的基础上进行了扩展，允许与 `any` 进行双向赋值，以及与具有对应数值的 `enum` 进行双向赋值。

根据具体情况，语言的不同位置会使用这两种兼容性机制之一。
出于实用目的，类型兼容性主要由赋值兼容性决定，即使在 `implements` 和 `extends` 子句中也是如此。

## `any`、`unknown`、`object`、`void`、`undefined`、`null` 和 `never` 的可赋值性

下表汇总了某些抽象类型之间的可赋值性。
行表示当前类型可以赋值给哪些类型，列表示哪些类型可以赋值给当前类型。
"<span class='black-tick'>✓</span>" 表示仅在关闭 [`strictNullChecks`](/tsconfig#strictNullChecks) 时兼容的组合。

<!-- This is the rendered form of https://github.com/microsoft/TypeScript-Website/pull/1490 -->
<table class="data">
<thead>
<tr>
<th></th>
<th align="center">any</th>
<th align="center">unknown</th>
<th align="center">object</th>
<th align="center">void</th>
<th align="center">undefined</th>
<th align="center">null</th>
<th align="center">never</th>
</tr>
</thead>
<tbody>
<tr>
<td>any →</td>
<td align="center"></td>
<td align="center"><span class="blue-tick" style="
    color: #007aff;
">✓</span></td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="red-cross">✕</span></td>
</tr>
<tr>
<td>unknown →</td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"></td>
<td align="center"><span class="red-cross">✕</span></td>
<td align="center"><span class="red-cross">✕</span></td>
<td align="center"><span class="red-cross">✕</span></td>
<td align="center"><span class="red-cross">✕</span></td>
<td align="center"><span class="red-cross">✕</span></td>
</tr>
<tr>
<td>object →</td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"></td>
<td align="center"><span class="red-cross">✕</span></td>
<td align="center"><span class="red-cross">✕</span></td>
<td align="center"><span class="red-cross">✕</span></td>
<td align="center"><span class="red-cross">✕</span></td>
</tr>
<tr>
<td>void →</td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="red-cross">✕</span></td>
<td align="center"></td>
<td align="center"><span class="red-cross">✕</span></td>
<td align="center"><span class="red-cross">✕</span></td>
<td align="center"><span class="red-cross">✕</span></td>
</tr>
<tr>
<td>undefined →</td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="black-tick">✓</span></td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"></td>
<td align="center"><span class="black-tick">✓</span></td>
<td align="center"><span class="red-cross">✕</span></td>
</tr>
<tr>
<td>null →</td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="black-tick">✓</span></td>
<td align="center"><span class="black-tick">✓</span></td>
<td align="center"><span class="black-tick">✓</span></td>
<td align="center"></td>
<td align="center"><span class="red-cross">✕</span></td>
</tr>
<tr>
<td>never →</td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"><span class="blue-tick">✓</span></td>
<td align="center"></td>
</tr>
</tbody>
</table>

重申[基础知识](/docs/handbook/2/basic-types.html)：

- 任何类型都可以赋值给自己。
- 就“哪些类型可以赋值给它们”而言，`any` 与 `unknown` 相同；不同之处在于，`unknown` 不能赋值给除 `any` 之外的任何类型。
- `unknown` 与 `never` 互为逆命题。
  任何类型都可以赋值给 `unknown`，`never` 可以赋值给任何类型。
  任何类型都不能赋值给 `never`，`unknown` 不能赋值给除 `any` 之外的任何类型。
- `void` 不能赋值给任何类型，也不能被任何类型赋值，但有以下例外：`any`、`unknown`、`never`、`undefined` 和 `null`（若 [`strictNullChecks`](/tsconfig#strictNullChecks) 关闭，详见表格）。
- 当 [`strictNullChecks`](/tsconfig#strictNullChecks) 关闭时，`null` 和 `undefined` 与 `never` 类似：可以赋值给大多数类型，但大多数类型不能赋值给它们。
  它们之间可以相互赋值。
- 当 [`strictNullChecks`](/tsconfig#strictNullChecks) 开启时，`null` 和 `undefined` 的行为更接近 `void`：不能赋值给任何类型，也不能被任何类型赋值，除了 `any`、`unknown` 和 `void`（`undefined` 始终可以赋值给 `void`）。
