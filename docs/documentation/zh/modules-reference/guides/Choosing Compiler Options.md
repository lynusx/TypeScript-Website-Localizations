---
title: 模块 - 选择编译器选项
short: 选择编译器选项
layout: docs
permalink: /zh/docs/handbook/modules/guides/choosing-compiler-options.html
oneline: 如何选择能够反映模块环境的编译器选项
translatable: true
---

## 我正在编写应用

单个 tsconfig.json 只能代表单一环境，无论是在可用的全局变量方面，还是在模块的行为方式方面。如果你的应用包含服务端代码、DOM 代码、Web Worker 代码、测试代码以及需要被所有这些代码共享的代码，那么每一部分都应该有其独立的 tsconfig.json，并通过[项目引用（Project References）](https://www.typescriptlang.org/docs/handbook/project-references.html#handbook-content)关联起来。然后，对每个 tsconfig.json 分别参考本指南进行配置。对于应用内类似库的项目（尤其是需要在多个运行时环境中运行的项目），请参阅“[我正在编写库](#im-writing-a-library)”一节。

### 我正在使用打包工具

除了采用以下配置之外，目前还建议在打包工具项目中**不要**设置 `{ "type": "module" }` 或使用 `.mts` 文件。[某些打包工具](https://andrewbranch.github.io/interop-test/#synthesizing-default-exports-for-cjs-modules)在这些情况下会采用不同的 ESM/CJS 互操作行为，而 TypeScript 目前在 `"moduleResolution": "bundler"` 下无法对此进行分析。有关更多信息，请参阅 [issue #54102](https://github.com/microsoft/TypeScript/issues/54102)。

```json5
{
  compilerOptions: {
    // 这不是一个完整的模板；它仅展示
    // 与模块相关的设置。
    // 请务必配置其他重要选项，
    // 如 `target`、`lib` 和 `strict`。

    // 必需
    module: 'esnext',
    moduleResolution: 'bundler',
    esModuleInterop: true,

    // 请查阅打包工具的文档
    customConditions: ['module'],

    // 推荐
    noEmit: true, // 或 `emitDeclarationOnly`
    allowImportingTsExtensions: true,
    allowArbitraryExtensions: true,
    verbatimModuleSyntax: true, // 或 `isolatedModules`
  },
}
```

### 我正在编译输出并在 Node.js 中运行

如果你打算生成 ES 模块，请记得设置 `"type": "module"` 或使用 `.mts` 文件。

```json5
{
  compilerOptions: {
    // 这不是一个完整的模板；它仅展示
    // 与模块相关的设置。
    // 请务必配置其他重要选项，
    // 如 `target`、`lib` 和 `strict`。

    // 必需
    module: 'nodenext',

    // 由 `"module": "nodenext"` 隐式启用：
    // "moduleResolution": "nodenext",
    // "esModuleInterop": true,
    // "target": "esnext",

    // 推荐
    verbatimModuleSyntax: true,
  },
}
```

### 我正在使用 ts-node

ts-node 致力于兼容可用于[在 Node.js 中编译和运行 JS 输出](#im-compiling-and-running-the-outputs-in-node)的相同代码与相同 tsconfig.json 配置。详情请参阅 [ts-node 文档](https://typestrong.org/ts-node/)。

### 我正在使用 tsx

ts-node 默认对 Node.js 的模块系统做出的修改极小，而 [tsx](https://github.com/esbuild-kit/tsx) 的行为则更接近打包工具，允许不带扩展名/目录索引的模块标识符，以及 ESM 和 CJS 的任意混用。针对 tsx，请采用与[打包工具相同的配置](#im-using-a-bundler)。

### 我正在为浏览器编写 ES 模块，不使用打包工具或模块编译器

TypeScript 目前没有专门针对该场景的选项，但你可以结合使用 `nodenext` 的 ESM 模块解析算法以及 `paths`（作为 URL 和 import map 支持的替代方案）来模拟该环境。

```json5
// tsconfig.json
{
  compilerOptions: {
    // 这不是一个完整的模板；它仅展示
    // 与模块相关的设置。
    // 请务必配置其他重要选项，
    // 如 `target`、`lib` 和 `strict`。

    // 与本地 package.json 中的 `"type": "module"` 配合使用时，
    // 这将强制在相对路径导入中包含文件扩展名。
    module: 'nodenext',
    paths: {
      // 将 TS 指向远程 URL 对应的本地类型：
      'https://esm.sh/lodash@4.17.21': [
        './node_modules/@types/lodash/index.d.ts',
      ],
      // 可选：将裸说明符导入指向一个空文件，
      // 以禁止从此处未列出的 node_modules 说明符导入：
      '*': ['./empty-file.ts'],
    },
  },
}
```

这种配置允许明确列出的 HTTPS 导入使用本地安装的类型声明文件，同时对通常会在 node_modules 中解析的导入报错：

```ts
import {} from 'lodash'
//             ^^^^^^^^
// 文件 '/project/empty-file.ts' 不是一个模块。ts(2306)
```

或者，你也可以使用 [import maps](https://github.com/WICG/import-maps) 在浏览器中明确将裸说明符（bare specifier）列表映射到 URL，同时依靠 `nodenext` 默认的 node_modules 查找机制或 `paths`，引导 TypeScript 找到这些裸说明符导入对应的类型声明文件：

```html
<script type="importmap">
  {
    "imports": {
      "lodash": "https://esm.sh/lodash@4.17.21"
    }
  }
</script>
```

```ts
import {} from 'lodash'
// 浏览器：https://esm.sh/lodash@4.17.21
// TypeScript：./node_modules/@types/lodash/index.d.ts
```

## 我正在编写库

<!-- TODO: I might move all this to a guide/appendix on library publishing and link -->

作为库作者选择编译配置，与作为应用开发者选择配置的过程有着本质的不同。在编写应用时，选择的配置旨在反映运行时环境或打包工具——这通常是一个行为明确的单一实体。而在编写库时，理想情况下应该在库使用者**所有可能的**编译配置下检查你的代码。但鉴于这并不现实，你可以转而采用最严格的配置，因为满足了最严格的配置往往就能满足其他所有配置。

```json5
{
  compilerOptions: {
    module: 'node18',
    target: 'es2020', // 设置为你所支持的 *最低* target
    strict: true,
    verbatimModuleSyntax: true,
    declaration: true,
    sourceMap: true,
    declarationMap: true,
    rootDir: 'src',
    outDir: 'dist',
  },
}
```

让我们来看看为什么选择其中的每一项配置：

- **`module: "node18"`**。当代码库与 Node.js 的模块系统兼容时，它几乎也总能正常运行于打包工具中。如果你使用第三方编译器生成 ESM 输出，请确保在 package.json 中设置了 `"type": "module"`，以便 TypeScript 将你的代码按 ESM 进行检查，因为 Node.js 中 ESM 的模块解析算法比 CommonJS 更为严格。作为示例，让我们看看如果库使用 `"moduleResolution": "bundler"` 进行编译会发生什么：

  ```ts
  export * from './utils'
  ```

  假设存在 `./utils.ts`（或 `./utils/index.ts`），打包工具处理这段代码没有任何问题，因此 `"moduleResolution": "bundler"` 不会报错。当使用 `"module": "esnext"` 编译时，该导出语句生成的 JavaScript 输出将与输入完全一致。如果该 JavaScript 被发布到 npm，使用打包工具的项目可以正常使用它，但在 Node.js 中运行时则会导致错误：

  ```
  Error [ERR_MODULE_NOT_FOUND]: Cannot find module '.../node_modules/dependency/utils' imported from .../node_modules/dependency/index.js
  Did you mean to import ./utils.js?
  ```

  另一方面，如果我们写成：

  ```ts
  export * from './utils.js'
  ```

  这将生成既能在 Node.js 中运行、**也能**在打包工具中运行的输出代码。

  简而言之，`"moduleResolution": "bundler"` 具有传染性，容易产生只能在打包工具中运行的代码。同样，`"moduleResolution": "nodenext"` 虽然只检查输出是否能在 Node.js 中运行，但在大多数情况下，能在 Node.js 中运行的模块代码在其他运行时和打包工具中也能正常工作。

- **`target: "es2020"`**。将该值设置为你打算支持的**最低** ECMAScript 版本，可确保生成的代码不会使用更高版本中引入的语言特性。由于 `target` 还会隐含相应的 `lib` 值，这也能确保你不会访问在较旧环境中可能不存在的全局变量。
- **`strict: true`**。如果不启用此选项，你编写的类型层面代码最终可能会出现在输出的 `.d.ts` 文件中，并在使用者开启 `strict` 编译时发生报错。例如，下面的 `extends` 子句：
  ```ts
  export interface Super {
    foo: string
  }
  export interface Sub extends Super {
    foo: string | undefined
  }
  ```
  只有在开启 `strictNullChecks` 时才会报错。另一方面，几乎不可能写出仅在**关闭** `strict` 时报错的代码，因此强烈建议库在编译时开启 `strict`。
- **`verbatimModuleSyntax: true`**。该选项可以防范几种可能会给库使用者带来困扰的模块相关陷阱。首先，它能防止写出根据使用者的 `esModuleInterop` 或 `allowSyntheticDefaultImports` 配置产生歧义解释的 import 语句。此前，人们常建议库在编译时不启用 `esModuleInterop`，因为在库中使用它可能会迫使使用者也必须开启它。然而，同样有可能写出**只有在未启用** `esModuleInterop` 时才能正常工作的导入，因此该选项的任意取值都无法保证库的可移植性。而 `verbatimModuleSyntax` 确实提供了这种保证。[^1] 其次，它能防止在将被输出为 CommonJS 的模块中使用 `export default`，因为这可能要求打包工具使用者和 Node.js ESM 使用者以不同的方式使用该模块。详情请参阅关于 [ESM/CJS 互操作性](/docs/handbook/modules/appendices/esm-cjs-interop.html#library-code-needs-special-considerations)的附录。
- **`declaration: true`** 会随同输出的 JavaScript 一起生成类型声明文件。库的使用者需要这些文件来获取类型信息。
- **`sourceMap: true`** 与 **`declarationMap: true`** 分别为输出的 JavaScript 和类型声明文件生成 source map。只有当库同时发布其源文件（`.ts`）时，它们才有意义。通过附带 source map 和源文件，库的使用者能相对更轻松地调试库代码。通过附带 declaration map 和源文件，使用者在对库的导入执行“跳转到定义（Go To Definition）”时能够直接查看原始的 TypeScript 源码。这两项配置都体现了开发体验与库体积之间的权衡，因此是否包含它们取决于你自己。
- **`rootDir: "src"`** 与 **`outDir: "dist"`**。使用单独的输出目录始终是一个好做法，而对于发布输入源文件的库来说更是**必不可少**。否则，[扩展名替换（extension substitution）](/docs/handbook/modules/reference.html#file-extension-substitution)会导致库的使用者加载库的 `.ts` 文件而非 `.d.ts` 文件，从而引发类型错误和性能问题。

### 打包库时的考量

如果你使用打包工具来生成库的输出，那么所有（未被外部化/externalized 的）导入都将由打包工具按照其明确的行为进行处理，而不是由用户不可预知的环境处理。在这种情况下，你可以使用 `"module": "esnext"` 和 `"moduleResolution": "bundler"`，但需注意以下两点：

1. 当部分文件被打包而部分文件被外部化时，TypeScript 无法对模块解析进行建模。在打包带有依赖项的库时，通常会将第一方库源码打包进单个文件中，但在打包输出中将外部依赖项的导入保留为真正的导入。这本质上意味着模块解析被拆分在打包工具与最终用户环境之间。要在 TypeScript 中模拟这一点，你会希望使用 `"moduleResolution": "bundler"` 处理被打包的导入，并使用 `"moduleResolution": "nodenext"`（或使用多种选项进行检查，以确保在各种最终用户环境中均可正常工作）处理被外部化的导入。但 TypeScript 无法配置为在同一次编译中使用两种不同的模块解析设置。因此，使用 `"moduleResolution": "bundler"` 可能会允许在打包工具中有效但在 Node.js 中不安全的外部依赖项导入。另一方面，使用 `"moduleResolution": "nodenext"` 则可能会对打包导入施加过于严格的要求。
2. 你必须确保声明文件也一同被打包。回顾[声明文件的第一法则](/docs/handbook/modules/theory.html#the-role-of-declaration-files)：每个声明文件精确代表一个 JavaScript 文件。如果你使用 `"moduleResolution": "bundler"` 并通过打包工具输出 ESM bundle，同时使用 `tsc` 生成众多独立的声明文件，那么在 `"module": "nodenext"` 下使用时，你的声明文件可能会引发错误。例如，如下输入文件：

   ```ts
   import { Component } from './extensionless-relative-import'
   ```

   其导入语句会被 JS 打包工具消除，但生成的声明文件中却包含完全相同的 import 语句。然而在 Node.js 中，该 import 语句将包含一个无效的模块标识符，因为它缺少文件扩展名。对于 Node.js 用户，TypeScript 会在声明文件上报错，并将引用 `Component` 的类型污染为 `any`（假定该依赖项在运行时会崩溃）。

   如果你的 TypeScript 打包工具不会生成打包后的声明文件，请使用 `"moduleResolution": "nodenext"`，以确保保留在声明文件中的导入与最终用户的 TypeScript 配置兼容。更进一步地说，甚至可以考虑不对你的库进行打包。

### 双重输出（Dual-emit）方案的注意事项

单次 TypeScript 编译（无论是输出文件还是仅进行类型检查）都假定每个输入文件只产生一个输出文件。即使 `tsc` 不输出任何内容，它对导入名称执行的类型检查也依赖于 tsconfig.json 中设置的模块与输出相关选项，借此获知输出文件在运行时的行为。虽然只要能配置 `tsc` 理解其他编译工具的输出内容，将第三方编译器与 `tsc` 类型检查结合使用通常是安全的，但任何只进行一次类型检查却输出两套不同模块格式产物的方案，都会导致（至少）其中一套输出产物未经检查。因为外部依赖项可能向 CommonJS 和 ESM 使用者暴露不同的 API，所以没有任何配置能保证在单次编译中两套输出都是类型安全的。在实践中，大多数依赖项都遵循最佳实践，双重输出产物通常能正常工作。在发布之前对所有输出 bundle 运行测试和[静态分析](https://npmjs.com/package/@arethetypeswrong/cli)，能显著降低严重问题未被察觉的风险。

[^1]: `verbatimModuleSyntax` 仅在 JS 编译器输出与 `tsc` 根据 tsconfig.json、源文件扩展名以及 package.json 中的 `"type"` 所推导出的相同模块类型时才有效。该选项的作用原理是强制保证手写的 `import`/`require` 与输出的 `import`/`require` 完全相同。任何从同一源文件同时生成 ESM 和 CJS 输出的配置在根本上都与 `verbatimModuleSyntax` 不兼容，因为该选项的全部目的就是防止你在任何会输出 `require` 的地方书写 `import`。如果将第三方编译器配置为输出与 `tsc` 不同的模块类型，`verbatimModuleSyntax` 也会失效——例如，在 tsconfig.json 中设置 `"module": "esnext"`，同时却配置 Babel 输出 CommonJS。
