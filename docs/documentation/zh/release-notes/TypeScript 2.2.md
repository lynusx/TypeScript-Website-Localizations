---
title: TypeScript 2.2
layout: docs
permalink: /zh/docs/handbook/release-notes/typescript-2-2.html
oneline: TypeScript 2.2 发布说明
---

## 混入类支持

TypeScript 2.2 新增了对 ECMAScript 2015 混入类模式的支持（详情请参阅 [MDN 混入说明](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Classes#Mix-ins) 和 [《使用 JavaScript 类实现"真正的"混入》](http://justinfagnani.com/2015/12/21/real-mixins-with-javascript-classes/)），同时新增了在交叉类型中将混入构造签名与普通构造签名合并的规则。

##### 首先明确一些术语

**混入构造函数类型**是指具有单个构造签名的类型，该签名接受一个 `any[]` 类型的剩余参数，并返回一个对象类型。例如，给定一个对象类型 `X`，`new (...args: any[]) => X` 就是一个实例类型为 `X` 的混入构造函数类型。

**混入类**是 `extends` 了某个类型参数类型的类声明或类表达式。混入类声明需满足以下规则：

- `extends` 表达式中类型参数的约束必须是混入构造函数类型。
- 混入类的构造函数（若有）必须有一个 `any[]` 类型的剩余参数，并且必须使用展开运算符将这些参数传递给 `super(...args)` 调用。

给定一个约束为 `X` 的参数化类型 `T` 的表达式 `Base`，混入类 `class C extends Base {...}` 在处理时等同于 `Base` 具有类型 `X`，其结果类型为交叉类型 `typeof C & T`。换言之，混入类表示为混入类构造函数类型与参数化基类构造函数类型的交叉。

在获取包含混入构造函数类型的交叉类型的构造签名时，混入构造签名会被丢弃，其实例类型会被混入交叉类型中其他构造签名的返回类型。例如，交叉类型 `{ new(...args: any[]) => A } & { new(s: string) => B }` 具有单个构造签名 `new(s: string) => A & B`。

##### 综合示例

```ts
class Point {
  constructor(
    public x: number,
    public y: number,
  ) {}
}

class Person {
  constructor(public name: string) {}
}

type Constructor<T> = new (...args: any[]) => T

function Tagged<T extends Constructor<{}>>(Base: T) {
  return class extends Base {
    _tag: string
    constructor(...args: any[]) {
      super(...args)
      this._tag = ''
    }
  }
}

const TaggedPoint = Tagged(Point)

let point = new TaggedPoint(10, 20)
point._tag = 'hello'

class Customer extends Tagged(Person) {
  accountBalance: number
}

let customer = new Customer('Joe')
customer._tag = 'test'
customer.accountBalance = 0
```

混入类可以通过在类型参数的约束中指定构造签名返回类型，来限制可混入的类的类型。例如，下面的 `WithLocation` 函数实现了一个子类工厂，它为所有满足 `Point` 接口（即拥有 `number` 类型的 `x` 和 `y` 属性）的类添加 `getLocation` 方法。

```ts
interface Point {
  x: number
  y: number
}

const WithLocation = <T extends Constructor<Point>>(Base: T) =>
  class extends Base {
    getLocation(): [number, number] {
      return [this.x, this.y]
    }
  }
```

## `object` 类型

TypeScript 此前没有专门表示非原始类型的类型，即不是 `number`、`string`、`boolean`、`symbol`、`null` 或 `undefined` 的任意类型。为此，新增了 `object` 类型。

有了 `object` 类型，`Object.create` 等 API 可以得到更准确的类型表示。例如：

```ts
declare function create(o: object | null): void

create({ prop: 0 }) // 正常
create(null) // 正常

create(42) // 错误
create('string') // 错误
create(false) // 错误
create(undefined) // 错误
```

## `new.target` 支持

`new.target` 元属性是 ES2015 引入的新语法。当通过 `new` 创建构造函数的实例时，`new.target` 的值会被设置为最初用于分配该实例的构造函数的引用。若函数是被直接调用而非通过 `new` 构造的，`new.target` 的值则为 `undefined`。

当需要在类的构造函数中设置 `Object.setPrototypeOf` 或 `__proto__` 时，`new.target` 非常有用。一个典型的使用场景是在 NodeJS v4 及更高版本中继承 `Error`。

##### 示例

```ts
class CustomError extends Error {
  constructor(message?: string) {
    super(message) // 'Error' 在此处打破了原型链
    Object.setPrototypeOf(this, new.target.prototype) // 恢复原型链
  }
}
```

生成的 JS 代码如下：

```js
var CustomError = (function (_super) {
  __extends(CustomError, _super)
  function CustomError() {
    var _newTarget = this.constructor
    var _this = _super.apply(this, arguments) // 'Error' 在此处打破了原型链
    _this.__proto__ = _newTarget.prototype // 恢复原型链
    return _this
  }
  return CustomError
})(Error)
```

`new.target` 在编写可构造函数时同样有用，例如：

```ts
function f() {
  if (new.target) {
    /* 通过 'new' 调用 */
  }
}
```

会被转译为：

```js
function f() {
  var _newTarget = this && this instanceof f ? this.constructor : void 0
  if (_newTarget) {
    /* 通过 'new' 调用 */
  }
}
```

## 改进表达式操作数中对 `null`/`undefined` 的检查

TypeScript 2.2 增强了对表达式中可空操作数的检查。具体来说，以下情况现在会被标记为错误：

- `+` 运算符的任一操作数为可空类型，且两个操作数均不为 `any` 或 `string` 类型。
- `-`、`*`、`**`、`/`、`%`、`<<`、`>>`、`>>>`、`&`、`|` 或 `^` 运算符的任一操作数为可空类型。
- `<`、`>`、`<=`、`>=` 或 `in` 运算符的任一操作数为可空类型。
- `instanceof` 运算符的右操作数为可空类型。
- `+`、`-`、`~`、`++` 或 `--` 一元运算符的操作数为可空类型。

若操作数的类型为 `null`、`undefined` 或包含 `null`/`undefined` 的联合类型，则认为该操作数是可空的。注意，联合类型情况仅在 [`strictNullChecks`](/tsconfig#strictNullChecks) 模式下出现，因为在经典类型检查模式中，`null` 和 `undefined` 会从联合类型中消失。

## 字符串索引签名类型的点运算符访问

具有字符串索引签名的类型可以使用 `[]` 记法进行索引访问，但此前不允许使用 `.` 记法。从 TypeScript 2.2 起，两种写法均被允许。

```ts
interface StringMap<T> {
  [x: string]: T
}

const map: StringMap<number>

map['prop1'] = 1
map.prop2 = 2
```

此规则仅适用于具有_显式_字符串索引签名的类型。对于没有索引签名的类型，使用 `.` 记法访问未知属性仍会报错。

## JSX 元素子节点的展开运算符支持

TypeScript 2.2 新增了对 JSX 元素子节点使用展开运算符的支持。详情请参阅 [facebook/jsx#57](https://github.com/facebook/jsx/issues/57)。

##### 示例

```ts
function Todo(prop: { key: number; todo: string }) {
  return <div>{prop.key.toString() + prop.todo}</div>;
}

function TodoList({ todos }: TodoListProps) {
  return (
    <div>{...todos.map(todo => <Todo key={todo.id} todo={todo.todo} />)}</div>
  );
}

let x: TodoListProps;

<TodoList {...x} />;
```

## 新增 `jsx: react-native`

React Native 的构建流程要求所有文件使用 `.js` 扩展名，即便文件中包含 JSX 语法。新增的 [`jsx`](/tsconfig#jsx) 值 `react-native` 会在输出文件中保留 JSX 语法，同时将扩展名设为 `.js`。
