---
title: 装饰器
layout: docs
permalink: /zh/docs/handbook/decorators.html
oneline: TypeScript 装饰器概览
translatable: true
---

> NOTE&nbsp; 本文档针对的是实验性的 Stage 2 装饰器实现。自 TypeScript 5.0 起已支持 Stage 3 装饰器。
> 参见：[TypeScript 5.0 中的装饰器](https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/#decorators)

## 介绍

随着 TypeScript 和 ES6 中引入了类（Class），在某些场景下需要额外的特性来支持为类及其成员添加注解或进行修改。
装饰器（Decorators）为类声明和成员提供了一种添加注解以及元编程（meta-programming）语法的途径。

> 延伸阅读（Stage 2）：[A Complete Guide to TypeScript Decorators](https://saul-mirone.github.io/a-complete-guide-to-typescript-decorator/)

若要启用对装饰器的实验性支持，你必须在命令行或 `tsconfig.json` 中启用 [`experimentalDecorators`](/tsconfig#experimentalDecorators) 编译器选项：

**命令行**：

```shell
tsc --target ES5 --experimentalDecorators
```

**tsconfig.json**：

```json tsconfig
{
  "compilerOptions": {
    "target": "ES5",
    "experimentalDecorators": true
  }
}
```

## 装饰器

_装饰器_（Decorator）是一种特殊类型的声明，它能够附加到[类声明](#class-decorators)、[方法](#method-decorators)、[访问器](#accessor-decorators)、[属性](#property-decorators)或[参数](#parameter-decorators)上。
装饰器采用 `@expression` 的形式，其中 `expression` 求值后必须为一个函数，该函数会在运行时被调用，并传入关于被装饰声明的信息。

例如，给定装饰器 `@sealed`，我们可以编写如下 `sealed` 函数：

```ts
function sealed(target) {
  // do something with 'target' ...
}
```

## 装饰器工厂

如果我们想要自定义装饰器如何应用于声明，可以编写一个装饰器工厂。
_装饰器工厂_（Decorator Factory）就是一个简单的函数，它返回一个将在运行时由装饰器调用的表达式。

我们可以按照如下方式编写装饰器工厂：

```ts
function color(value: string) {
  // this is the decorator factory, it sets up
  // the returned decorator function
  return function (target) {
    // this is the decorator
    // do something with 'target' and 'value'...
  }
}
```

## 装饰器组合

多个装饰器可以同时应用到同一个声明上，例如写在同一行：

```ts twoslash
// @experimentalDecorators
// @noErrors
function f() {}
function g() {}
// ---cut---
@f @g x
```

或者写在多行：

```ts twoslash
// @experimentalDecorators
// @noErrors
function f() {}
function g() {}
// ---cut---
@f
@g
x
```

当多个装饰器应用于同一个声明时，其求值方式类似于[数学中的复合函数](https://wikipedia.org/wiki/Function_composition)。在该模型中，复合函数 _f_ 和 _g_ 时，得到的复合结果 (_f_ ∘ _g_)(_x_) 等价于 _f_(_g_(_x_))。

因此，在 TypeScript 中对单个声明上的多个装饰器求值时，会执行以下步骤：

1. 各个装饰器的表达式从上到下依次求值。
2. 求值结果随后作为函数从下到上依次调用。

如果使用[装饰器工厂](#decorator-factories)，我们可以通过以下示例观察到这种求值顺序：

<!-- prettier-ignore -->
```ts twoslash
// @experimentalDecorators
function first() {
  console.log("first(): factory evaluated");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("first(): called");
  };
}

function second() {
  console.log("second(): factory evaluated");
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("second(): called");
  };
}

class ExampleClass {
  @first()
  @second()
  method() {}
}
```

这将在控制台中输出如下内容：

```shell
first(): factory evaluated
second(): factory evaluated
second(): called
first(): called
```

## 装饰器求值

类中各种声明所应用的装饰器具有明确规定的执行顺序：

1. 对于每个实例成员，依次应用*参数装饰器*，接着是*方法*、*访问器*或*属性装饰器*。
2. 对于每个静态成员，依次应用*参数装饰器*，接着是*方法*、*访问器*或*属性装饰器*。
3. 对于构造函数，应用*参数装饰器*。
4. 对于类，应用*类装饰器*。

## 类装饰器

*类装饰器*声明在类声明的正前方。
类装饰器应用于类的构造函数，可用于观察、修改或替换类定义。
类装饰器不能在声明文件或任何其他外部上下文（例如 `declare` 类）中使用。

类装饰器的表达式将在运行时作为函数被调用，被装饰类的构造函数将作为其唯一参数传入。

如果类装饰器返回了一个值，它将使用所提供的构造函数来替换原有的类声明。

> NOTE&nbsp; 如果你选择返回一个新的构造函数，必须自行注意维护原始的原型（prototype）。
> 运行时应用装饰器的逻辑**不会**自动为你处理这一点。

以下是将类装饰器（`@sealed`）应用于 `BugReport` 类的示例：

```ts twoslash
// @experimentalDecorators
function sealed(constructor: Function) {
  Object.seal(constructor)
  Object.seal(constructor.prototype)
}
// ---cut---
@sealed
class BugReport {
  type = 'report'
  title: string

  constructor(t: string) {
    this.title = t
  }
}
```

我们可以使用如下函数声明来定义 `@sealed` 装饰器：

```ts
function sealed(constructor: Function) {
  Object.seal(constructor)
  Object.seal(constructor.prototype)
}
```

当 `@sealed` 执行时，它会同时封闭（seal）构造函数及其原型，从而阻止在运行时通过访问 `BugReport.prototype` 或在 `BugReport` 自身上定义属性来为此类添加或移除任何功能（请注意，ES2015 的类本质上只是基于原型的构造函数的语法糖）。该装饰器**不会**阻止其他类继承 `BugReport`。

接下来是一个关于如何重写构造函数以设置新默认值的示例：

<!-- prettier-ignore -->
```ts twoslash
// @errors: 2339
// @experimentalDecorators
function reportableClassDecorator<T extends { new (...args: any[]): {} }>(constructor: T) {
  return class extends constructor {
    reportingURL = "http://www...";
  };
}

@reportableClassDecorator
class BugReport {
  type = "report";
  title: string;

  constructor(t: string) {
    this.title = t;
  }
}

const bug = new BugReport("Needs dark mode");
console.log(bug.title); // Prints "Needs dark mode"
console.log(bug.type); // Prints "report"

// Note that the decorator _does not_ change the TypeScript type
// and so the new property `reportingURL` is not known
// to the type system:
bug.reportingURL;
```

## 方法装饰器

*方法装饰器*声明在方法声明的正前方。
该装饰器应用于方法的*属性描述符*（Property Descriptor），可用于观察、修改或替换方法定义。
方法装饰器不能在声明文件、重载或任何其他外部上下文（例如 `declare` 类）中使用。

方法装饰器的表达式将在运行时作为函数被调用，并传入以下三个参数：

1. 对于静态成员是类的构造函数，对于实例成员则是类的原型。
2. 成员的名称。
3. 该成员的*属性描述符*。

> NOTE&emsp; 如果你的脚本编译目标（script target）低于 `ES5`，*属性描述符*将为 `undefined`。

如果方法装饰器返回了一个值，该值将被用作该方法的*属性描述符*。

> NOTE&emsp; 如果你的脚本编译目标低于 `ES5`，返回值将被忽略。

以下是将方法装饰器（`@enumerable`）应用于 `Greeter` 类中方法的示例：

<!-- prettier-ignore -->
```ts twoslash
// @experimentalDecorators
function enumerable(value: boolean) {
  return function (target: any,propertyKey: string,descriptor: PropertyDescriptor) {
    descriptor.enumerable = value;
  };
}
// ---cut---
class Greeter {
  greeting: string;
  constructor(message: string) {
    this.greeting = message;
  }

  @enumerable(false)
  greet() {
    return "Hello, " + this.greeting;
  }
}
```

我们可以使用如下函数声明来定义 `@enumerable` 装饰器：

<!-- prettier-ignore -->
```ts twoslash
function enumerable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.enumerable = value;
  };
}
```

这里的 `@enumerable(false)` 装饰器是一个[装饰器工厂](#decorator-factories)。
当 `@enumerable(false)` 装饰器被调用时，它会修改属性描述符的 `enumerable` 属性。

## 访问器装饰器

*访问器装饰器*声明在访问器声明的正前方。
访问器装饰器应用于访问器的*属性描述符*，可用于观察、修改或替换访问器的定义。
访问器装饰器不能在声明文件或任何其他外部上下文（例如 `declare` 类）中使用。

> NOTE&emsp; TypeScript 不允许同时为一个成员的 `get` 和 `set` 访问器添加装饰器。
> 相反，该成员的所有装饰器必须应用到按文档顺序排列的第一个访问器上。
> 这是因为装饰器应用于*属性描述符*，而属性描述符同时结合了 `get` 和 `set` 访问器，并非分别针对每个声明。

访问器装饰器的表达式将在运行时作为函数被调用，并传入以下三个参数：

1. 对于静态成员是类的构造函数，对于实例成员则是类的原型。
2. 成员的名称。
3. 该成员的*属性描述符*。

> NOTE&emsp; 如果你的脚本编译目标低于 `ES5`，*属性描述符*将为 `undefined`。

如果访问器装饰器返回了一个值，该值将被用作该成员的*属性描述符*。

> NOTE&emsp; 如果你的脚本编译目标低于 `ES5`，返回值将被忽略。

以下是将访问器装饰器（`@configurable`）应用于 `Point` 类成员的示例：

```ts twoslash
// @experimentalDecorators
function configurable(value: boolean) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor,
  ) {
    descriptor.configurable = value
  }
}
// ---cut---
class Point {
  private _x: number
  private _y: number
  constructor(x: number, y: number) {
    this._x = x
    this._y = y
  }

  @configurable(false)
  get x() {
    return this._x
  }

  @configurable(false)
  get y() {
    return this._y
  }
}
```

我们可以使用如下函数声明来定义 `@configurable` 装饰器：

<!-- prettier-ignore -->
```ts
function configurable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.configurable = value;
  };
}
```

## 属性装饰器

*属性装饰器*声明在属性声明的正前方。
属性装饰器不能在声明文件或任何其他外部上下文（例如 `declare` 类）中使用。

属性装饰器的表达式将在运行时作为函数被调用，并传入以下两个参数：

1. 对于静态成员是类的构造函数，对于实例成员则是类的原型。
2. 成员的名称。

> NOTE&emsp; 由于 TypeScript 中属性装饰器的初始化方式，属性装饰器不会收到*属性描述符*作为参数。
> 这是因为在定义原型成员时，目前没有机制来描述实例属性，也无法观察或修改属性的初始化器。其返回值同样会被忽略。
> 因此，属性装饰器只能用来观察类中是否声明了特定名称的属性。

我们可以利用这一信息来记录有关属性的元数据，如下例所示：

```ts
class Greeter {
  @format('Hello, %s')
  greeting: string

  constructor(message: string) {
    this.greeting = message
  }

  greet() {
    let formatString = getFormat(this, 'greeting')
    return formatString.replace('%s', this.greeting)
  }
}
```

然后，我们可以使用如下函数声明来定义 `@format` 装饰器和 `getFormat` 函数：

```ts
import 'reflect-metadata'

const formatMetadataKey = Symbol('format')

function format(formatString: string) {
  return Reflect.metadata(formatMetadataKey, formatString)
}

function getFormat(target: any, propertyKey: string) {
  return Reflect.getMetadata(formatMetadataKey, target, propertyKey)
}
```

这里的 `@format("Hello, %s")` 装饰器是一个[装饰器工厂](#decorator-factories)。
当调用 `@format("Hello, %s")` 时，它会使用 `reflect-metadata` 库中的 `Reflect.metadata` 函数为该属性添加一条元数据条目。
当调用 `getFormat` 时，它会读取该格式的元数据值。

> NOTE&emsp; 该示例需要依赖 `reflect-metadata` 库。
> 有关 `reflect-metadata` 库的更多信息，请参阅[元数据](#metadata)。

## 参数装饰器

*参数装饰器*声明在参数声明的正前方。
参数装饰器应用于类构造函数或方法声明的函数。
参数装饰器不能在声明文件、重载或任何其他外部上下文（例如 `declare` 类）中使用。

参数装饰器的表达式将在运行时作为函数被调用，并传入以下三个参数：

1. 对于静态成员是类的构造函数，对于实例成员则是类的原型。
2. 成员的名称。
3. 参数在函数参数列表中的序数索引。

> NOTE&emsp; 参数装饰器只能用来观察方法上是否声明了某个参数。

参数装饰器的返回值会被忽略。

以下是将参数装饰器（`@required`）应用于 `BugReport` 类成员参数的示例：

<!-- prettier-ignore -->
```ts twoslash
// @experimentalDecorators
function validate(target: any, propertyName: string, descriptor: TypedPropertyDescriptor<any>) {}
function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {}
// ---cut---
class BugReport {
  type = "report";
  title: string;

  constructor(t: string) {
    this.title = t;
  }

  @validate
  print(@required verbose: boolean) {
    if (verbose) {
      return `type: ${this.type}\ntitle: ${this.title}`;
    } else {
     return this.title; 
    }
  }
}
```

然后，我们可以使用如下函数声明来定义 `@required` 和 `@validate` 装饰器：

<!-- prettier-ignore -->
```ts twoslash
// @experimentalDecorators
// @emitDecoratorMetadata
import "reflect-metadata";
const requiredMetadataKey = Symbol("required");

function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {
  let existingRequiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyKey) || [];
  existingRequiredParameters.push(parameterIndex);
  Reflect.defineMetadata( requiredMetadataKey, existingRequiredParameters, target, propertyKey);
}

function validate(target: any, propertyName: string, descriptor: TypedPropertyDescriptor<Function>) {
  let method = descriptor.value!;

  descriptor.value = function () {
    let requiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyName);
    if (requiredParameters) {
      for (let parameterIndex of requiredParameters) {
        if (parameterIndex >= arguments.length || arguments[parameterIndex] === undefined) {
          throw new Error("Missing required argument.");
        }
      }
    }
    return method.apply(this, arguments);
  };
}
```

`@required` 装饰器会添加一条元数据条目，将该参数标记为必填。
随后，`@validate` 装饰器会将现有的 `print` 方法包装在一个函数中，该函数会在调用原始方法之前验证参数。

> NOTE&emsp; 该示例需要依赖 `reflect-metadata` 库。
> 有关 `reflect-metadata` 库的更多信息，请参阅[元数据](#metadata)。

## 元数据

部分示例使用了 `reflect-metadata` 库，该库为[实验性元数据 API](https://github.com/rbuckton/ReflectDecorators) 添加了 polyfill。
该库目前尚未成为 ECMAScript (JavaScript) 标准的一部分。
然而，一旦装饰器被正式采纳为 ECMAScript 标准的一部分，这些扩展也将被提议采纳。

你可以通过 npm 安装该库：

```shell
npm i reflect-metadata --save
```

TypeScript 包含了对带有装饰器的声明生成某些类型元数据的实验性支持。
若要启用该实验性支持，你必须在命令行或 `tsconfig.json` 中设置 [`emitDecoratorMetadata`](/tsconfig#emitDecoratorMetadata) 编译器选项：

**命令行**：

```shell
tsc --target ES5 --experimentalDecorators --emitDecoratorMetadata
```

**tsconfig.json**：

```json tsconfig
{
  "compilerOptions": {
    "target": "ES5",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

启用后，只要导入了 `reflect-metadata` 库，额外的设计期类型信息就会在运行时暴露出来。

我们可以在以下示例中看到实际应用：

<!-- prettier-ignore -->
```ts twoslash
// @emitDecoratorMetadata
// @experimentalDecorators
// @strictPropertyInitialization: false
import "reflect-metadata";

class Point {
  constructor(public x: number, public y: number) {}
}

class Line {
  private _start: Point;
  private _end: Point;

  @validate
  set start(value: Point) {
    this._start = value;
  }

  get start() {
    return this._start;
  }

  @validate
  set end(value: Point) {
    this._end = value;
  }

  get end() {
    return this._end;
  }
}

function validate<T>(target: any, propertyKey: string, descriptor: TypedPropertyDescriptor<T>) {
  let set = descriptor.set!;
  
  descriptor.set = function (value: T) {
    let type = Reflect.getMetadata("design:type", target, propertyKey);

    if (!(value instanceof type)) {
      throw new TypeError(`Invalid type, got ${typeof value} not ${type.name}.`);
    }

    set.call(this, value);
  };
}

const line = new Line()
line.start = new Point(0, 0)

// @ts-ignore
// line.end = {}

// Fails at runtime with:
// > Invalid type, got object not Point

```

TypeScript 编译器将使用 `@Reflect.metadata` 装饰器注入设计期类型信息。
你可以将其视为等价于以下 TypeScript 代码：

```ts
class Line {
  private _start: Point
  private _end: Point

  @validate
  @Reflect.metadata('design:type', Point)
  set start(value: Point) {
    this._start = value
  }
  get start() {
    return this._start
  }

  @validate
  @Reflect.metadata('design:type', Point)
  set end(value: Point) {
    this._end = value
  }
  get end() {
    return this._end
  }
}
```

> NOTE&emsp; 装饰器元数据是一项实验性特性，可能会在未来的版本中引入破坏性变更。
