---
title: MSBuild 中的编译器选项
layout: docs
permalink: /zh/docs/handbook/compiler-options-in-msbuild.html
oneline: MSBuild 项目中可用的编译器选项。
---

## 概述

当你在基于 MSBuild 的项目（例如 ASP.NET Core 项目）中使用 TypeScript 时，可以通过两种方式配置 TypeScript：通过 `tsconfig.json` 或通过项目设置。

## 使用 `tsconfig.json`

我们建议在可能的情况下为项目使用 `tsconfig.json`。若要在现有项目中添加该文件，请在现代版本的 Visual Studio 中向项目添加一个名为“TypeScript JSON 配置文件”的新项。

新增的 `tsconfig.json` 将作为文件列表和配置等 TypeScript 专用构建信息的唯一真实来源（source of truth）。你可以在此处了解[关于 TSConfig 工作原理的说明](/docs/handbook/tsconfig-json.html)，此处还提供了[完整的参考手册](/tsconfig)。

## 使用项目设置

你也可以在项目设置中定义 TypeScript 的配置。这是通过在 `.csproj` 文件中编辑 XML 来定义描述构建行为的 `PropertyGroup` 实现的：

```xml
<PropertyGroup>
  <TypeScriptNoEmitOnError>true</TypeScriptNoEmitOnError>
  <TypeScriptNoImplicitReturns>true</TypeScriptNoImplicitReturns>
</PropertyGroup>
```

常见的 TypeScript 设置有一系列映射关系，这些设置直接映射到 [TypeScript CLI 选项](/docs/handbook/compiler-options.html)，用于帮助你编写更易理解的项目文件。你可以使用 [TSConfig 参考手册](/tsconfig)获取关于每个映射的可用值及默认值的更多信息。

<!-- Start of replacement  --><h3>CLI 映射</h3>

  <table class='cli-option' width="100%">
    <thead>
    <tr>
    <th>MSBuild 配置名称</th>
    <th>TSC 标志</th>
    </tr>
  </thead>
  <tbody>

<tr class='odd' name='allowJs'>
<td><code>&#x3C;TypeScriptAllowJS&#x3E;</code></td>
<td><code><a href='/tsconfig/#allowJs'>--allowJs</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>允许 JavaScript 文件成为程序的一部分。使用 <code>checkJS</code> 选项可从这些文件中获取报错信息。</p>

</tr></td>
<tr class='even' name='removeComments'>
<td><code>&#x3C;TypeScriptRemoveComments&#x3E;</code></td>
<td><code><a href='/tsconfig/#removeComments'>--removeComments</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>禁止生成注释。</p>

</tr></td>
<tr class='odd' name='noImplicitAny'>
<td><code>&#x3C;TypeScriptNoImplicitAny&#x3E;</code></td>
<td><code><a href='/tsconfig/#noImplicitAny'>--noImplicitAny</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>为具有隐式 <code>any</code> 类型的表达式和声明启用错误报告。</p>

</tr></td>
<tr class='even' name='declaration'>
<td><code>&#x3C;TypeScriptGeneratesDeclarations&#x3E;</code></td>
<td><code><a href='/tsconfig/#declaration'>--declaration</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>为项目中的 TypeScript 和 JavaScript 文件生成 .d.ts 文件。</p>

</tr></td>
<tr class='odd' name='module'>
<td><code>&#x3C;TypeScriptModuleKind&#x3E;</code></td>
<td><code><a href='/tsconfig/#module'>--module</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定生成什么模块代码。</p>

</tr></td>
<tr class='even' name='jsx'>
<td><code>&#x3C;TypeScriptJSXEmit&#x3E;</code></td>
<td><code><a href='/tsconfig/#jsx'>--jsx</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定生成何种 JSX 代码。</p>

</tr></td>
<tr class='odd' name='outDir'>
<td><code>&#x3C;TypeScriptOutDir&#x3E;</code></td>
<td><code><a href='/tsconfig/#outDir'>--outDir</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>为所有生成的文件指定输出目录。</p>

</tr></td>
<tr class='even' name='sourcemap'>
<td><code>&#x3C;TypeScriptSourceMap&#x3E;</code></td>
<td><code><a href='/tsconfig/#sourcemap'>--sourcemap</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>为生成的 JavaScript 文件创建 source map 文件。</p>

</tr></td>
<tr class='odd' name='target'>
<td><code>&#x3C;TypeScriptTarget&#x3E;</code></td>
<td><code><a href='/tsconfig/#target'>--target</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>设置生成的 JavaScript 语言版本，并包含兼容的库声明。</p>

</tr></td>
<tr class='even' name='noResolve'>
<td><code>&#x3C;TypeScriptNoResolve&#x3E;</code></td>
<td><code><a href='/tsconfig/#noResolve'>--noResolve</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>禁止通过 <code>import</code>、<code>require</code> 或 <code>&#x3C;reference></code> 增加 TypeScript 应添加到项目中的文件数量。</p>

</tr></td>
<tr class='odd' name='mapRoot'>
<td><code>&#x3C;TypeScriptMapRoot&#x3E;</code></td>
<td><code><a href='/tsconfig/#mapRoot'>--mapRoot</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定调试器查找 source map 文件的位置，而不是使用生成时的相对位置。</p>

</tr></td>
<tr class='even' name='sourceRoot'>
<td><code>&#x3C;TypeScriptSourceRoot&#x3E;</code></td>
<td><code><a href='/tsconfig/#sourceRoot'>--sourceRoot</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定调试器查找引用源文件的根路径。</p>

</tr></td>
<tr class='odd' name='charset'>
<td><code>&#x3C;TypeScriptCharset&#x3E;</code></td>
<td><code><a href='/tsconfig/#charset'>--charset</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>已不再支持。在早期版本中，用于手动指定读取文件时的文本编码。</p>

</tr></td>
<tr class='even' name='emitBOM'>
<td><code>&#x3C;TypeScriptEmitBOM&#x3E;</code></td>
<td><code><a href='/tsconfig/#emitBOM'>--emitBOM</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在输出文件的开头写入 UTF-8 字节顺序标记（BOM）。</p>

</tr></td>
<tr class='odd' name='noLib'>
<td><code>&#x3C;TypeScriptNoLib&#x3E;</code></td>
<td><code><a href='/tsconfig/#noLib'>--noLib</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>禁用包含任何库文件，包括默认的 lib.d.ts。</p>

</tr></td>
<tr class='even' name='preserveConstEnums'>
<td><code>&#x3C;TypeScriptPreserveConstEnums&#x3E;</code></td>
<td><code><a href='/tsconfig/#preserveConstEnums'>--preserveConstEnums</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>禁止在生成的代码中擦除 <code>const enum</code> 声明。</p>

</tr></td>
<tr class='odd' name='suppressImplicitAnyIndexErrors'>
<td><code>&#x3C;TypeScriptSuppressImplicitAnyIndexErrors&#x3E;</code></td>
<td><code><a href='/tsconfig/#suppressImplicitAnyIndexErrors'>--suppressImplicitAnyIndexErrors</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>在对缺少索引签名的对象进行索引时，抑制 <code>noImplicitAny</code> 错误。</p>

</tr></td>
<tr class='even' name='noEmitHelpers'>
<td><code>&#x3C;TypeScriptNoEmitHelpers&#x3E;</code></td>
<td><code><a href='/tsconfig/#noEmitHelpers'>--noEmitHelpers</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>禁用在编译输出中生成类似 <code>__extends</code> 的自定义辅助函数。</p>

</tr></td>
<tr class='odd' name='inlineSourceMap'>
<td><code>&#x3C;TypeScriptInlineSourceMap&#x3E;</code></td>
<td><code><a href='/tsconfig/#inlineSourceMap'>--inlineSourceMap</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>在生成的 JavaScript 文件中内联包含 sourcemap 内容。</p>

</tr></td>
<tr class='even' name='inlineSources'>
<td><code>&#x3C;TypeScriptInlineSources&#x3E;</code></td>
<td><code><a href='/tsconfig/#inlineSources'>--inlineSources</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>将源代码作为内联内容包含在生成的 JavaScript 中的 source map 内。</p>

</tr></td>
<tr class='odd' name='newLine'>
<td><code>&#x3C;TypeScriptNewLine&#x3E;</code></td>
<td><code><a href='/tsconfig/#newLine'>--newLine</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>设置生成文件时的换行字符。</p>

</tr></td>
<tr class='even' name='isolatedModules'>
<td><code>&#x3C;TypeScriptIsolatedModules&#x3E;</code></td>
<td><code><a href='/tsconfig/#isolatedModules'>--isolatedModules</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>确保每个文件都能安全地转译，而无需依赖其他导入。</p>

</tr></td>
<tr class='odd' name='emitDecoratorMetadata'>
<td><code>&#x3C;TypeScriptEmitDecoratorMetadata&#x3E;</code></td>
<td><code><a href='/tsconfig/#emitDecoratorMetadata'>--emitDecoratorMetadata</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>为源文件中带装饰器的声明生成设计类型元数据。</p>

</tr></td>
<tr class='even' name='rootDir'>
<td><code>&#x3C;TypeScriptRootDir&#x3E;</code></td>
<td><code><a href='/tsconfig/#rootDir'>--rootDir</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定源文件中的根目录。</p>

</tr></td>
<tr class='odd' name='experimentalDecorators'>
<td><code>&#x3C;TypeScriptExperimentalDecorators&#x3E;</code></td>
<td><code><a href='/tsconfig/#experimentalDecorators'>--experimentalDecorators</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>启用对 TC39 stage 2 草案装饰器的实验性支持。</p>

</tr></td>
<tr class='even' name='moduleResolution'>
<td><code>&#x3C;TypeScriptModuleResolution&#x3E;</code></td>
<td><code><a href='/tsconfig/#moduleResolution'>--moduleResolution</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定 TypeScript 如何根据给定的模块说明符查找文件。</p>

</tr></td>
<tr class='odd' name='suppressExcessPropertyErrors'>
<td><code>&#x3C;TypeScriptSuppressExcessPropertyErrors&#x3E;</code></td>
<td><code><a href='/tsconfig/#suppressExcessPropertyErrors'>--suppressExcessPropertyErrors</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>禁用在创建对象字面量时报告额外属性错误。</p>

</tr></td>
<tr class='even' name='reactNamespace'>
<td><code>&#x3C;TypeScriptReactNamespace&#x3E;</code></td>
<td><code><a href='/tsconfig/#reactNamespace'>--reactNamespace</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定调用 <code>createElement</code> 的对象。这仅在针对 <code>react</code> JSX 输出时适用。</p>

</tr></td>
<tr class='odd' name='skipDefaultLibCheck'>
<td><code>&#x3C;TypeScriptSkipDefaultLibCheck&#x3E;</code></td>
<td><code><a href='/tsconfig/#skipDefaultLibCheck'>--skipDefaultLibCheck</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>跳过对 TypeScript 内置的 .d.ts 文件的类型检查。</p>

</tr></td>
<tr class='even' name='allowUnusedLabels'>
<td><code>&#x3C;TypeScriptAllowUnusedLabels&#x3E;</code></td>
<td><code><a href='/tsconfig/#allowUnusedLabels'>--allowUnusedLabels</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>禁用未使用的标签的错误报告。</p>

</tr></td>
<tr class='odd' name='noImplicitReturns'>
<td><code>&#x3C;TypeScriptNoImplicitReturns&#x3E;</code></td>
<td><code><a href='/tsconfig/#noImplicitReturns'>--noImplicitReturns</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>对函数中未显式 return 的代码分支报错。</p>

</tr></td>
<tr class='even' name='noFallthroughCasesInSwitch'>
<td><code>&#x3C;TypeScriptNoFallthroughCasesInSwitch&#x3E;</code></td>
<td><code><a href='/tsconfig/#noFallthroughCasesInSwitch'>--noFallthroughCasesInSwitch</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>对 switch 语句中贯穿（fallthrough）的 case 分支启用错误报告。</p>

</tr></td>
<tr class='odd' name='allowUnreachableCode'>
<td><code>&#x3C;TypeScriptAllowUnreachableCode&#x3E;</code></td>
<td><code><a href='/tsconfig/#allowUnreachableCode'>--allowUnreachableCode</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>禁用无法访问的代码的错误报告。</p>

</tr></td>
<tr class='even' name='forceConsistentCasingInFileNames'>
<td><code>&#x3C;TypeScriptForceConsistentCasingInFileNames&#x3E;</code></td>
<td><code><a href='/tsconfig/#forceConsistentCasingInFileNames'>--forceConsistentCasingInFileNames</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>确保导入语句中的文件名大小写完全一致。</p>

</tr></td>
<tr class='odd' name='allowSyntheticDefaultImports'>
<td><code>&#x3C;TypeScriptAllowSyntheticDefaultImports&#x3E;</code></td>
<td><code><a href='/tsconfig/#allowSyntheticDefaultImports'>--allowSyntheticDefaultImports</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>当模块没有默认导出时，允许使用 'import x from y'。</p>

</tr></td>
<tr class='even' name='noImplicitUseStrict'>
<td><code>&#x3C;TypeScriptNoImplicitUseStrict&#x3E;</code></td>
<td><code><a href='/tsconfig/#noImplicitUseStrict'>--noImplicitUseStrict</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>禁止在生成的 JavaScript 文件中添加 'use strict' 指令。</p>

</tr></td>
<tr class='odd' name='lib'>
<td><code>&#x3C;TypeScriptLib&#x3E;</code></td>
<td><code><a href='/tsconfig/#lib'>--lib</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定一组描述目标运行时环境的内置库声明文件。</p>

</tr></td>
<tr class='even' name='baseUrl'>
<td><code>&#x3C;TypeScriptBaseUrl&#x3E;</code></td>
<td><code><a href='/tsconfig/#baseUrl'>--baseUrl</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定用于解析裸模块说明符的基础目录。</p>

</tr></td>
<tr class='odd' name='declarationDir'>
<td><code>&#x3C;TypeScriptDeclarationDir&#x3E;</code></td>
<td><code><a href='/tsconfig/#declarationDir'>--declarationDir</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定所生成声明文件的输出目录。</p>

</tr></td>
<tr class='even' name='noImplicitThis'>
<td><code>&#x3C;TypeScriptNoImplicitThis&#x3E;</code></td>
<td><code><a href='/tsconfig/#noImplicitThis'>--noImplicitThis</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>当 <code>this</code> 表达式具有隐式 <code>any</code> 类型时启用错误报告。</p>

</tr></td>
<tr class='odd' name='skipLibCheck'>
<td><code>&#x3C;TypeScriptSkipLibCheck&#x3E;</code></td>
<td><code><a href='/tsconfig/#skipLibCheck'>--skipLibCheck</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>跳过对所有 .d.ts 声明文件的类型检查。</p>

</tr></td>
<tr class='even' name='strictNullChecks'>
<td><code>&#x3C;TypeScriptStrictNullChecks&#x3E;</code></td>
<td><code><a href='/tsconfig/#strictNullChecks'>--strictNullChecks</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在进行类型检查时考虑 <code>null</code> 和 <code>undefined</code>。</p>

</tr></td>
<tr class='odd' name='noUnusedLocals'>
<td><code>&#x3C;TypeScriptNoUnusedLocals&#x3E;</code></td>
<td><code><a href='/tsconfig/#noUnusedLocals'>--noUnusedLocals</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>当局部变量未被读取时启用报错。</p>

</tr></td>
<tr class='even' name='noUnusedParameters'>
<td><code>&#x3C;TypeScriptNoUnusedParameters&#x3E;</code></td>
<td><code><a href='/tsconfig/#noUnusedParameters'>--noUnusedParameters</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>当函数参数未被读取时报错。</p>

</tr></td>
<tr class='odd' name='alwaysStrict'>
<td><code>&#x3C;TypeScriptAlwaysStrict&#x3E;</code></td>
<td><code><a href='/tsconfig/#alwaysStrict'>--alwaysStrict</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>确保始终输出 'use strict'。</p>

</tr></td>
<tr class='even' name='importHelpers'>
<td><code>&#x3C;TypeScriptImportHelpers&#x3E;</code></td>
<td><code><a href='/tsconfig/#importHelpers'>--importHelpers</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>允许在每个项目中从 tslib 统一导入辅助函数，而不是在每个文件中重复包含它们。</p>

</tr></td>
<tr class='odd' name='jsxFactory'>
<td><code>&#x3C;TypeScriptJSXFactory&#x3E;</code></td>
<td><code><a href='/tsconfig/#jsxFactory'>--jsxFactory</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定生成 React JSX 代码时使用的 JSX 工厂函数，例如 'React.createElement' 或 'h'。</p>

</tr></td>
<tr class='even' name='stripInternal'>
<td><code>&#x3C;TypeScriptStripInternal&#x3E;</code></td>
<td><code><a href='/tsconfig/#stripInternal'>--stripInternal</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>禁止为 JSDoc 注释中带有 <code>@internal</code> 的代码生成声明。</p>

</tr></td>
<tr class='odd' name='checkJs'>
<td><code>&#x3C;TypeScriptCheckJs&#x3E;</code></td>
<td><code><a href='/tsconfig/#checkJs'>--checkJs</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>在进行类型检查的 JavaScript 文件中启用错误报告。</p>

</tr></td>
<tr class='even' name='downlevelIteration'>
<td><code>&#x3C;TypeScriptDownlevelIteration&#x3E;</code></td>
<td><code><a href='/tsconfig/#downlevelIteration'>--downlevelIteration</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>为迭代操作生成更符合规范，但代码更繁琐且性能较低的 JavaScript。</p>

</tr></td>
<tr class='odd' name='strict'>
<td><code>&#x3C;TypeScriptStrict&#x3E;</code></td>
<td><code><a href='/tsconfig/#strict'>--strict</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>启用所有严格类型检查选项。</p>

</tr></td>
<tr class='even' name='noStrictGenericChecks'>
<td><code>&#x3C;TypeScriptNoStrictGenericChecks&#x3E;</code></td>
<td><code><a href='/tsconfig/#noStrictGenericChecks'>--noStrictGenericChecks</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>禁用函数类型中泛型签名的严格检查。</p>

</tr></td>
<tr class='odd' name='preserveSymlinks'>
<td><code>&#x3C;TypeScriptPreserveSymlinks&#x3E;</code></td>
<td><code><a href='/tsconfig/#preserveSymlinks'>--preserveSymlinks</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>禁止将符号链接解析为其真实路径。这与 Node.js 中的同名标志相对应。</p>

</tr></td>
<tr class='even' name='strictFunctionTypes'>
<td><code>&#x3C;TypeScriptStrictFunctionTypes&#x3E;</code></td>
<td><code><a href='/tsconfig/#strictFunctionTypes'>--strictFunctionTypes</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在对函数赋值时，检查确保参数和返回值满足子类型兼容性。</p>

</tr></td>
<tr class='odd' name='strictPropertyInitialization'>
<td><code>&#x3C;TypeScriptStrictPropertyInitialization&#x3E;</code></td>
<td><code><a href='/tsconfig/#strictPropertyInitialization'>--strictPropertyInitialization</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>检查类中已声明但在构造函数中未明确初始化的属性。</p>

</tr></td>
<tr class='even' name='esModuleInterop'>
<td><code>&#x3C;TypeScriptESModuleInterop&#x3E;</code></td>
<td><code><a href='/tsconfig/#esModuleInterop'>--esModuleInterop</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>生成额外的 JavaScript 代码以更轻松地支持导入 CommonJS 模块。这同时会启用 <code>allowSyntheticDefaultImports</code> 以实现类型兼容。</p>

</tr></td>
<tr class='odd' name='emitDeclarationOnly'>
<td><code>&#x3C;TypeScriptEmitDeclarationOnly&#x3E;</code></td>
<td><code><a href='/tsconfig/#emitDeclarationOnly'>--emitDeclarationOnly</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>仅输出 .d.ts 声明文件，而不输出 JavaScript 文件。</p>

</tr></td>
<tr class='even' name='keyofStringsOnly'>
<td><code>&#x3C;TypeScriptKeyofStringsOnly&#x3E;</code></td>
<td><code><a href='/tsconfig/#keyofStringsOnly'>--keyofStringsOnly</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>让 keyof 仅返回 string，而非 string、number 或 symbol。已废弃的历史选项。</p>

</tr></td>
<tr class='odd' name='useDefineForClassFields'>
<td><code>&#x3C;TypeScriptUseDefineForClassFields&#x3E;</code></td>
<td><code><a href='/tsconfig/#useDefineForClassFields'>--useDefineForClassFields</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>输出符合 ECMAScript 标准规范的类字段。</p>

</tr></td>
<tr class='even' name='declarationMap'>
<td><code>&#x3C;TypeScriptDeclarationMap&#x3E;</code></td>
<td><code><a href='/tsconfig/#declarationMap'>--declarationMap</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>为 d.ts 文件创建 source map。</p>

</tr></td>
<tr class='odd' name='resolveJsonModule'>
<td><code>&#x3C;TypeScriptResolveJsonModule&#x3E;</code></td>
<td><code><a href='/tsconfig/#resolveJsonModule'>--resolveJsonModule</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>允许导入 .json 文件。</p>

</tr></td>
<tr class='even' name='strictBindCallApply'>
<td><code>&#x3C;TypeScriptStrictBindCallApply&#x3E;</code></td>
<td><code><a href='/tsconfig/#strictBindCallApply'>--strictBindCallApply</a></code></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>检查 <code>bind</code>、<code>call</code> 和 <code>apply</code> 方法的参数是否与原函数相匹配。</p>

</tr></td>
<tr class='odd' name='noEmitOnError'>
<td><code>&#x3C;TypeScriptNoEmitOnError&#x3E;</code></td>
<td><code><a href='/tsconfig/#noEmitOnError'>--noEmitOnError</a></code></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>报告任何类型检查错误时不生成文件。</p>

</tr></td>
</tbody></table>
<!-- End of replacement  -->

### 附加标志

由于 MSBuild 系统会将参数直接传递给 TypeScript CLI，因此你可以使用 `TypeScriptAdditionalFlags` 选项来提供上面未包含映射的具体标志。

例如，以下配置将启用 [`noPropertyAccessFromIndexSignature`](/tsconfig#noPropertyAccessFromIndexSignature)：

```xml
<TypeScriptAdditionalFlags> $(TypeScriptAdditionalFlags) --noPropertyAccessFromIndexSignature</TypeScriptAdditionalFlags>
```

### Debug 和 Release 构建

你可以使用 PropertyGroup 条件来定义不同的配置集。例如，在生产环境中剥离注释和 source map 是一项常见任务。在以下示例中，我们定义了具有不同 TypeScript 配置的 Debug 和 Release 属性组：

```xml
<PropertyGroup Condition="'$(Configuration)' == 'Debug'">
  <TypeScriptRemoveComments>false</TypeScriptRemoveComments>
  <TypeScriptSourceMap>true</TypeScriptSourceMap>
</PropertyGroup>

<PropertyGroup Condition="'$(Configuration)' == 'Release'">
  <TypeScriptRemoveComments>true</TypeScriptRemoveComments>
  <TypeScriptSourceMap>false</TypeScriptSourceMap>
</PropertyGroup>

<Import
    Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets"
    Condition="Exists('$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets')" />
```

### ToolsVersion

项目文件中的 `<TypeScriptToolsVersion>1.7</TypeScriptToolsVersion>` 属性值指定了用于构建的编译器版本（在本例中为 1.7）。
这样可以确保项目在不同机器上使用相同版本的编译器进行构建。

如果未指定 `TypeScriptToolsVersion`，则将使用机器上安装的最新编译器版本进行构建。

使用较新版本 TS 的用户在首次加载项目时将看到升级项目的提示。

### TypeScriptCompileBlocked

如果你使用其他构建工具来构建项目（例如 Gulp、Grunt 等），同时使用 VS 获取开发和调试体验，请在项目中设置 `<TypeScriptCompileBlocked>true</TypeScriptCompileBlocked>`。
这样可以在按下 F5 时只提供全部编辑支持，而不会触发构建。

### TypeScriptEnableIncrementalMSBuild（TypeScript 4.2 Beta 及更高版本）

默认情况下，MSBuild 会尝试仅在自上次编译以来项目的源文件发生更新时才运行 TypeScript 编译器。
然而，如果这种行为导致了问题（例如启用了 TypeScript 的 [`incremental`](/tsconfig#incremental) 选项时），可以设置 `<TypeScriptEnableIncrementalMSBuild>false</TypeScriptEnableIncrementalMSBuild>`，以确保在每次运行 MSBuild 时都调用 TypeScript 编译器。
