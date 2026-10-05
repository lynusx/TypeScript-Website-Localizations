---
display: 'Isolated Modules'
oneline: '确保每个文件都能安全地转译，而无需依赖其他导入。'
---

除了使用 TypeScript 将 TypeScript 代码转译为 JavaScript 代码外，通常也会使用如 [Babel](https://babeljs.io) 等其他转译器来完成这项工作。
然而，其他转译器一次只能处理单个文件，这意味着它们无法应用依赖于完整类型系统理解的代码转换。
这一限制同样适用于被某些构建工具使用的 TypeScript `ts.transpileModule` API。

这些局限性可能会在使用某些 TypeScript 特性（如 `const enum` 和 `namespace`）时导致运行时问题。
开启 `isolatedModules` 标志会提示 TypeScript：当你编写了某些无法被单文件转译过程正确解析的代码时发出警告。

它不会改变代码的运行行为，也不会以其他方式改变 TypeScript 的检查与发射（emit）流程。

以下是启用 `isolatedModules` 时无法正常工作的一些代码示例。

#### 导出非值标识符

在 TypeScript 中，你可以导入一个类型（_type_），随后再将其导出：

```ts twoslash
// @noErrors
import { someType, someFunction } from 'someModule'

someFunction()

export { someType, someFunction }
```

由于 `someType` 并不存在对应的值，生成的 `export` 不会尝试导出它（否则在 JavaScript 中会导致运行时错误）：

```js
export { someFunction }
```

单文件转译器无法获知 `someType` 是否产生了值，因此导出仅指向类型的名称属于错误行为。

#### 非模块文件

如果启用了 `isolatedModules`，命名空间（namespace）仅允许在模块（_modules_，即包含某种形式的 `import`/`export`）中使用。如果在非模块文件中出现命名空间，则会报错：

```ts twoslash
// @errors: 1277
// @isolatedModules
namespace Instantiated {
  export const x = 1
}
```

此限制不适用于 `.d.ts` 文件。

#### 引用 `const enum` 成员

在 TypeScript 中，当你引用 `const enum` 的成员时，该引用会在生成的 JavaScript 中被替换为其具体的实际值。将以下 TypeScript：

```ts twoslash
declare const enum Numbers {
  Zero = 0,
  One = 1,
}
console.log(Numbers.Zero + Numbers.One)
```

转换为如下 JavaScript：

```ts twoslash
// @showEmit
// @removeComments
declare const enum Numbers {
  Zero = 0,
  One = 1,
}
console.log(Numbers.Zero + Numbers.One)
```

在不了解这些成员具体值的情况下，其他转译器无法替换对 `Numbers` 的引用，若保留不管则会导致运行时错误（因为在运行时根本不存在 `Numbers` 对象）。
因此，当启用 `isolatedModules` 时，引用环境（ambient）`const enum` 成员属于错误行为。
