---
display: 'Verbatim Module Syntax'
oneline: "不转换也不省略任何未标记为仅类型的导入或导出，确保根据 'module' 设置将其写入输出文件的格式中。"
---

默认情况下，TypeScript 会执行名为 *导入省略*（import elision）的操作。
大体上，如果你编写类似如下代码：

```ts
import { Car } from './car'

export function drive(car: Car) {
  // ...
}
```

TypeScript 会检测到你仅将该导入用作类型，从而彻底丢弃该导入。
输出的 JavaScript 大致如下：

```js
export function drive(car) {
  // ...
}
```

大多数情况下这样处理很合适，因为如果 `Car` 不是从 `./car` 导出的值，在运行时就会抛出错误。

但在某些边界情况下，这会增加一层复杂性。
例如，请注意这里并没有类似 `import "./car";` 的语句 —— 该导入被完全丢弃了。
对于是否存在副作用（side-effects）的模块而言，这实际上会带来很大差异。

TypeScript 针对 JavaScript 的生成策略还存在其他几层复杂性 —— 导入省略并不总是仅取决于导入的使用方式，通常还会参考值的声明方式。
因此，像下面这样的代码：

```ts
export { Car } from './car'
```

究竟应该保留还是丢弃并不总是很明确。
如果 `Car` 是使用类似 `class` 声明的，则可以将其保留在生成的 JavaScript 文件中。
但如果 `Car` 仅被声明为 `type` 别名或 `interface`，那么生成的 JavaScript 文件根本不应该导出 `Car`。

虽然 TypeScript 或许能够根据跨文件的信息做出这些生成决策，但并非所有编译器都能做到这一点。

导入和导出上的 `type` 修饰符在一定程度上缓解了这些情况。
我们可以通过使用 `type` 修饰符，明确指出某个导入或导出仅用于类型分析，并可在 JavaScript 文件中完全丢弃。

```ts
// This statement can be dropped entirely in JS output
import type * as car from './car'

// The named import/export 'Car' can be dropped in JS output
import { type Car } from './car'
export { type Car } from './car'
```

单纯依靠 `type` 修饰符并不完全起效 —— 默认情况下，模块省略仍然会丢弃导入，而且没有任何机制强制区分 `type` 与普通导入导出。
因此，TypeScript 提供了 `--importsNotUsedAsValues` 标志以确保使用 `type` 修饰符，提供了 `--preserveValueImports` 以防止*某些*模块省略行为，以及提供了 `--isolatedModules` 以确保 TypeScript 代码能在不同编译器之间正常运行。
遗憾的是，理清这 3 个标志的细节非常困难，且仍然存在一些具有出乎意料行为的边界情况。

TypeScript 5.0 引入了一个名为 `--verbatimModuleSyntax` 的新选项来简化这种情况。
其规则简单得多 —— 任何没有 `type` 修饰符的导入或导出都会被原样保留。
任何使用了 `type` 修饰符的内容都会被彻底丢弃。

```ts
// Erased away entirely.
import type { A } from 'a'

// Rewritten to 'import { b } from "bcd";'
import { b, type c, type d } from 'bcd'

// Rewritten to 'import {} from "xyz";'
import { type xyz } from 'xyz'
```

有了这个新选项，真正做到了所见即所得。

不过，在涉及模块互操作性（module interop）时，这确实会产生一些影响。
在该标志下，当你的设置或文件扩展名暗指不同的模块系统时，ECMAScript `import` 和 `export` 不会被重写为 `require` 调用。
相反，你会收到报错。
如果你需要生成使用 `require` 和 `module.exports` 的代码，则必须使用早于 ES2015 的 TypeScript 模块语法：

<table>
<thead>
    <tr>
        <th>Input TypeScript</th>
        <th>Output JavaScript</th>
    </tr>
</thead>

<tr>
<td>

```ts
import foo = require('foo')
```

</td>
<td>

```js
const foo = require('foo')
```

</td>
</tr>
<tr>
<td>

```ts
function foo() {}
function bar() {}
function baz() {}

export = {
  foo,
  bar,
  baz,
}
```

</td>
<td>

```js
function foo() {}
function bar() {}
function baz() {}

module.exports = {
  foo,
  bar,
  baz,
}
```

</td>
</tr>
</table>

虽然这是一项限制，但它确实有助于让某些问题更加显而易见。
例如，在 `--module node16` 下，很容易忘记在 [`package.json` 中设置 `type` 字段](https://nodejs.org/api/packages.html#type)。
这会导致开发者在不经意间编写 CommonJS 模块而非 ES 模块，从而产生出人意料的查找规则和 JavaScript 输出。
这个新标志能确保你清楚明确地使用所需的文件类型，因为两者在语法上有意进行了区分。

由于 `--verbatimModuleSyntax` 提供了比 `--importsNotUsedAsValues` 和 `--preserveValueImports` 更一致的使用体验，因此弃用了这两个现有标志，转而推荐使用它。

有关更多详细信息，请阅读[原始 Pull Request](https://github.com/microsoft/TypeScript/pull/52203) 和[其提案 Issue](https://github.com/microsoft/TypeScript/issues/51479)。
