---
display: 'Erasable Syntax Only'
oneline: '禁止使用不属于 ECMAScript 的运行时语法结构。'
---

从 v23.6 开始，Node.js [支持直接运行 TypeScript 文件](https://nodejs.org/api/typescript.html#type-stripping)；
但是，在该模式下仅支持没有运行时语义的 TypeScript 特有语法。
换句话说，必须能够轻松地从文件中*擦除*任何 TypeScript 特有语法，并留下一个合法的 JavaScript 文件。

这意味着不支持以下语法结构：

- `enum` 声明
- 包含运行时代码的 `namespace` 和 `module`
- 类中的参数属性（parameter properties）
- 非 ECMAScript 标准的 `import =` 和 `export =` 赋值
- `<prefix>` 前缀风格的类型断言

```ts
// ❌ error: An `import ... = require(...)` alias
import foo = require('foo')

// ❌ error: A namespace with runtime code.
namespace container {
  foo.method()

  export type Bar = string
}

// ❌ error: An `import =` alias
import Bar = container.Bar

class Point {
  // ❌ error: Parameter properties
  constructor(
    public x: number,
    public y: number,
  ) {}
}

// ❌ error: An `export =` assignment.
export = Point

// ❌ error: An enum declaration.
enum Direction {
  Up,
  Down,
  Left,
  Right,
}

// ❌ error: <prefix>-style type assertion.
const num = <number>1
```

类似的工具如 [ts-blank-space](https://github.com/bloomberg/ts-blank-space) 或 [Amaro](https://github.com/nodejs/amaro)（Node.js 中类型擦除的底层库）也有相同的限制。
如果遇到不符合这些要求的代码，这些工具会提供有用的错误提示，但直到实际尝试运行时你才会发现代码无法工作。

`--erasableSyntaxOnly` 标志会让 TypeScript 在遇到大多数具有运行时行为的 TypeScript 特有语法结构时报错。

```ts
class C {
    constructor(public x: number) { }
    //          ~~~~~~~~~~~~~~~~
    // error! This syntax is not allowed when 'erasableSyntaxOnly' is enabled.
    }
}
```

通常，你会希望将该标志与 `--verbatimModuleSyntax` 结合使用，后者可以确保模块包含恰当的导入语法，并且不会发生导入省略（import elision）。
