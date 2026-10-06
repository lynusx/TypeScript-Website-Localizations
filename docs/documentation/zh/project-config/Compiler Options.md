---
title: tsc CLI 选项
layout: docs
permalink: /zh/docs/handbook/compiler-options.html
oneline: tsc CLI 编译器选项的概要总览
disable_toc: true
---

## 使用 CLI

在本地运行 `tsc` 将编译由最近的 `tsconfig.json` 定义的项目，或者你也可以通过传入所需文件的 glob 来编译一组 TypeScript 文件。当在命令行中指定了输入文件时，`tsconfig.json` 文件将被忽略。

```sh
# 基于在文件系统中向后查找 tsconfig.json 来执行编译
tsc

# 仅使用编译器默认设置针对 index.ts 输出 JS
tsc index.ts

# 使用默认设置针对 src 文件夹中的所有 .ts 文件输出 JS
tsc src/*.ts

# 使用 tsconfig.production.json 中的编译器设置输出引用的文件
tsc --project tsconfig.production.json

# 为 js 文件输出 d.ts 文件，同时显示布尔类型的编译器选项
tsc index.js --declaration --emitDeclarationOnly

# 通过接受字符串参数的编译器选项，从两个文件输出单个 .js 文件
tsc app.ts util.ts --target esnext --outfile index.js
```

## 编译器选项

**如果你正在寻找有关 tsconfig 中编译器选项的更多信息，请查看 [TSConfig 参考](/tsconfig)**

<!-- Start of replacement  --><h3>CLI 命令</h3>
<table class="cli-option" width="100%">
  <thead>
    <tr>
      <th>标志</th>
      <th>类型</th>
    </tr>
  </thead>
  <tbody>
<tr class='odd' name='all'>
  <td><code>--all</code></td>
  <td><p><code>boolean</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>显示所有编译器选项。</p>
</td></tr>

<tr class='even' name='help'>
  <td><code>--help</code></td>
  <td><p><code>boolean</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>提供 CLI 本地帮助信息。</p>
</td></tr>

<tr class='odd' name='ignoreConfig'>
  <td><code>--ignoreConfig</code></td>
  <td><p><code>boolean</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>忽略找到的 tsconfig，并使用命令行选项和文件进行构建。</p>
</td></tr>

<tr class='even' name='init'>
  <td><code>--init</code></td>
  <td><p><code>boolean</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>初始化一个 TypeScript 项目并创建 tsconfig.json 文件。</p>
</td></tr>

<tr class='odd' name='listFilesOnly'>
  <td><code>--listFilesOnly</code></td>
  <td><p><code>boolean</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>打印参与编译的文件名称，然后停止处理。</p>
</td></tr>

<tr class='even' name='locale'>
  <td><code>--locale</code></td>
  <td><p><code>string</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>设置 TypeScript 输出消息的语言。此选项不影响代码生成（emit）。</p>
</td></tr>

<tr class='odd' name='project'>
  <td><code>--project</code></td>
  <td><p><code>string</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>根据给定的配置文件路径或包含 'tsconfig.json' 的文件夹路径编译项目。</p>
</td></tr>

<tr class='even' name='showConfig'>
  <td><code>--showConfig</code></td>
  <td><p><code>boolean</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>打印最终配置而不是执行构建。</p>
</td></tr>

<tr class='odd' name='version'>
  <td><code>--version</code></td>
  <td><p><code>boolean</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>打印编译器的版本。</p>
</td></tr>

</tbody></table>

<h3>构建选项</h3>
<table class="cli-option" width="100%">
  <thead>
    <tr>
      <th>标志</th>
      <th>类型</th>
    </tr>
  </thead>
  <tbody>
<tr class='odd' name='build'>
  <td><code>--build</code></td>
  <td><p><code>boolean</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>构建一个或多个项目及其依赖项（若已过期）。</p>
</td></tr>

<tr class='even' name='clean'>
  <td><code>--clean</code></td>
  <td><p><code>boolean</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>删除所有项目的输出内容。</p>
</td></tr>

<tr class='odd' name='dry'>
  <td><code>--dry</code></td>
  <td><p><code>boolean</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>显示将要构建的内容（如果指定了 '--clean'，则显示将要删除的内容）。</p>
</td></tr>

<tr class='even' name='force'>
  <td><code><a href='/tsconfig/#force'>--force</a></code></td>
  <td><p><code>boolean</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>构建所有项目，包括那些看似已是最新的项目。</p>
</td></tr>

<tr class='odd' name='verbose'>
  <td><code><a href='/tsconfig/#verbose'>--verbose</a></code></td>
  <td><p><code>boolean</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>启用详细日志记录。</p>
</td></tr>

</tbody></table>

<h3>Watch 选项</h3>
<table class="cli-option" width="100%">
  <thead>
    <tr>
      <th>标志</th>
      <th>类型</th>
    </tr>
  </thead>
  <tbody>
<tr class='odd' name='excludeDirectories'>
  <td><code><a href='/tsconfig/#excludeDirectories'>--excludeDirectories</a></code></td>
  <td><p><code>list</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>从监视流程中排除一组目录。</p>
</td></tr>

<tr class='even' name='excludeFiles'>
  <td><code><a href='/tsconfig/#excludeFiles'>--excludeFiles</a></code></td>
  <td><p><code>list</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在监视模式的处理中排除指定的文件列表。</p>
</td></tr>

<tr class='odd' name='fallbackPolling'>
  <td><code><a href='/tsconfig/#fallbackPolling'>--fallbackPolling</a></code></td>
  <td><p><code>fixedinterval</code>, <code>priorityinterval</code>, <code>dynamicpriority</code>, or <code>fixedchunksize</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定在系统耗尽原生文件监视器时，监视器应使用的后备策略。</p>
</td></tr>

<tr class='even' name='synchronousWatchDirectory'>
  <td><code><a href='/tsconfig/#synchronousWatchDirectory'>--synchronousWatchDirectory</a></code></td>
  <td><p><code>boolean</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在原生不支持递归监视的平台上，同步调用回调并更新目录监视器的状态。</p>
</td></tr>

<tr class='odd' name='watch'>
  <td><code>--watch</code></td>
  <td><p><code>boolean</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>监视输入文件。</p>
</td></tr>

<tr class='even' name='watchDirectory'>
  <td><code><a href='/tsconfig/#watchDirectory'>--watchDirectory</a></code></td>
  <td><p><code>usefsevents</code>, <code>fixedpollinginterval</code>, <code>dynamicprioritypolling</code>, or <code>fixedchunksizepolling</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在缺乏递归文件监视功能的系统上，指定如何监视目录。</p>
</td></tr>

<tr class='odd' name='watchFile'>
  <td><code><a href='/tsconfig/#watchFile'>--watchFile</a></code></td>
  <td><p><code>fixedpollinginterval</code>, <code>prioritypollinginterval</code>, <code>dynamicprioritypolling</code>, <code>fixedchunksizepolling</code>, <code>usefsevents</code>, or <code>usefseventsonparentdirectory</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定 TypeScript 监视（watch）模式的工作方式。</p>
</td></tr>

</tbody></table>

<h3>编译器标志</h3>
<table class="cli-option" width="100%">
  <thead>
    <tr>
      <th>标志</th>
      <th>类型</th>
      <th>默认值</th>
    </tr>
  </thead>
  <tbody>
<tr class='odd' name='allowArbitraryExtensions'>
  <td><code><a href='/tsconfig/#allowArbitraryExtensions'>--allowArbitraryExtensions</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>允许导入具有任意扩展名的文件（只要存在对应的声明文件）。</p>
</td></tr>

<tr class='even' name='allowImportingTsExtensions'>
  <td><code><a href='/tsconfig/#allowImportingTsExtensions'>--allowImportingTsExtensions</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#rewriteRelativeImportExtensions"><code>rewriteRelativeImportExtensions</code></a>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>允许导入语句中包含 TypeScript 文件扩展名。</p>
</td></tr>

<tr class='odd' name='allowJs'>
  <td><code><a href='/tsconfig/#allowJs'>--allowJs</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code>, unless <code>checkJs</code> is set</p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>允许 JavaScript 文件成为程序的一部分。使用 <code>checkJS</code> 选项可从这些文件中获取报错信息。</p>
</td></tr>

<tr class='even' name='allowSyntheticDefaultImports'>
  <td><code><a href='/tsconfig/#allowSyntheticDefaultImports'>--allowSyntheticDefaultImports</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#esModuleInterop"><code>esModuleInterop</code></a> is enabled, <a href="#module"><code>module</code></a> is <code>system</code>, or <a href="#module-resolution"><code>moduleResolution</code></a> is <code>bundler</code>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>当模块没有默认导出时，允许使用 'import x from y'。</p>
</td></tr>

<tr class='odd' name='allowUmdGlobalAccess'>
  <td><code><a href='/tsconfig/#allowUmdGlobalAccess'>--allowUmdGlobalAccess</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>允许在模块内部访问 UMD 全局变量。</p>
</td></tr>

<tr class='even' name='allowUnreachableCode'>
  <td><code><a href='/tsconfig/#allowUnreachableCode'>--allowUnreachableCode</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>禁用无法访问的代码的错误报告。</p>
</td></tr>

<tr class='odd' name='allowUnusedLabels'>
  <td><code><a href='/tsconfig/#allowUnusedLabels'>--allowUnusedLabels</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>禁用未使用的标签的错误报告。</p>
</td></tr>

<tr class='even' name='alwaysStrict'>
  <td><code><a href='/tsconfig/#alwaysStrict'>--alwaysStrict</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#strict"><code>strict</code></a>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>确保始终输出 'use strict'。</p>
</td></tr>

<tr class='odd' name='assumeChangesOnlyAffectDirectDependencies'>
  <td><code><a href='/tsconfig/#assumeChangesOnlyAffectDirectDependencies'>--assumeChangesOnlyAffectDirectDependencies</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>在使用 <a href="#incremental"><code>incremental</code></a> 和 <code>watch</code> 模式的项目中，假定文件内部的变更只会影响直接依赖该文件的文件。</p>
</td></tr>

<tr class='even' name='baseUrl'>
  <td><code><a href='/tsconfig/#baseUrl'>--baseUrl</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定用于解析裸模块说明符的基础目录。</p>
</td></tr>

<tr class='odd' name='charset'>
  <td><code><a href='/tsconfig/#charset'>--charset</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td><p><code>utf8</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>已不再支持。在早期版本中，用于手动指定读取文件时的文本编码。</p>
</td></tr>

<tr class='even' name='checkJs'>
  <td><code><a href='/tsconfig/#checkJs'>--checkJs</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在进行类型检查的 JavaScript 文件中启用错误报告。</p>
</td></tr>

<tr class='odd' name='composite'>
  <td><code><a href='/tsconfig/#composite'>--composite</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>启用相关约束，以便 TypeScript 项目可以配合项目引用（Project References）使用。</p>
</td></tr>

<tr class='even' name='customConditions'>
  <td><code><a href='/tsconfig/#customConditions'>--customConditions</a></code></td>
  <td><p><code>list</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>解析导入时，在解析器特定默认条件之外额外设置的条件。</p>
</td></tr>

<tr class='odd' name='declaration'>
  <td><code><a href='/tsconfig/#declaration'>--declaration</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#composite"><code>composite</code></a>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>为项目中的 TypeScript 和 JavaScript 文件生成 .d.ts 文件。</p>
</td></tr>

<tr class='even' name='declarationDir'>
  <td><code><a href='/tsconfig/#declarationDir'>--declarationDir</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定所生成声明文件的输出目录。</p>
</td></tr>

<tr class='odd' name='declarationMap'>
  <td><code><a href='/tsconfig/#declarationMap'>--declarationMap</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>为 d.ts 文件创建 Source Map。</p>
</td></tr>

<tr class='even' name='diagnostics'>
  <td><code><a href='/tsconfig/#diagnostics'>--diagnostics</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在构建后输出编译器性能信息。</p>
</td></tr>

<tr class='odd' name='disableReferencedProjectLoad'>
  <td><code><a href='/tsconfig/#disableReferencedProjectLoad'>--disableReferencedProjectLoad</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>减少 TypeScript 自动加载的项目数量。</p>
</td></tr>

<tr class='even' name='disableSizeLimit'>
  <td><code><a href='/tsconfig/#disableSizeLimit'>--disableSizeLimit</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>解除 TypeScript 语言服务中对 JavaScript 文件源码总大小 20MB 的上限限制。</p>
</td></tr>

<tr class='odd' name='disableSolutionSearching'>
  <td><code><a href='/tsconfig/#disableSolutionSearching'>--disableSolutionSearching</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>在编辑时让项目不参与多项目引用检查。</p>
</td></tr>

<tr class='even' name='disableSourceOfProjectReferenceRedirect'>
  <td><code><a href='/tsconfig/#disableSourceOfProjectReferenceRedirect'>--disableSourceOfProjectReferenceRedirect</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在引用组合项目时，禁用优先使用源文件而非声明文件的行为。</p>
</td></tr>

<tr class='odd' name='downlevelIteration'>
  <td><code><a href='/tsconfig/#downlevelIteration'>--downlevelIteration</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>为迭代操作生成更符合规范，但代码更繁琐且性能较低的 JavaScript。</p>
</td></tr>

<tr class='even' name='emitBOM'>
  <td><code><a href='/tsconfig/#emitBOM'>--emitBOM</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在输出文件的开头写入 UTF-8 字节顺序标记（BOM）。</p>
</td></tr>

<tr class='odd' name='emitDeclarationOnly'>
  <td><code><a href='/tsconfig/#emitDeclarationOnly'>--emitDeclarationOnly</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>仅输出 .d.ts 声明文件，而不输出 JavaScript 文件。</p>
</td></tr>

<tr class='even' name='emitDecoratorMetadata'>
  <td><code><a href='/tsconfig/#emitDecoratorMetadata'>--emitDecoratorMetadata</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>为源文件中带装饰器的声明生成设计类型元数据。</p>
</td></tr>

<tr class='odd' name='erasableSyntaxOnly'>
  <td><code><a href='/tsconfig/#erasableSyntaxOnly'>--erasableSyntaxOnly</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>禁止使用不属于 ECMAScript 的运行时语法结构。</p>
</td></tr>

<tr class='even' name='esModuleInterop'>
  <td><code><a href='/tsconfig/#esModuleInterop'>--esModuleInterop</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#module"><code>module</code></a> is <code>node16</code>, <code>nodenext</code>, or <code>preserve</code>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>生成额外的 JavaScript 代码以更轻松地支持导入 CommonJS 模块。这同时会启用 <a href="#allowSyntheticDefaultImports"><code>allowSyntheticDefaultImports</code></a> 以实现类型兼容。</p>
</td></tr>

<tr class='odd' name='exactOptionalPropertyTypes'>
  <td><code><a href='/tsconfig/#exactOptionalPropertyTypes'>--exactOptionalPropertyTypes</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>按字面定义解释可选属性类型，而不是自动添加 <code>undefined</code>。</p>
</td></tr>

<tr class='even' name='experimentalDecorators'>
  <td><code><a href='/tsconfig/#experimentalDecorators'>--experimentalDecorators</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>启用对 TC39 stage 2 草案装饰器的实验性支持。</p>
</td></tr>

<tr class='odd' name='explainFiles'>
  <td><code><a href='/tsconfig/#explainFiles'>--explainFiles</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>打印在编译期间读取的文件以及将其包含在内的原因。</p>
</td></tr>

<tr class='even' name='extendedDiagnostics'>
  <td><code><a href='/tsconfig/#extendedDiagnostics'>--extendedDiagnostics</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在构建后输出更详细的编译器性能信息。</p>
</td></tr>

<tr class='odd' name='forceConsistentCasingInFileNames'>
  <td><code><a href='/tsconfig/#forceConsistentCasingInFileNames'>--forceConsistentCasingInFileNames</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>确保导入语句中的文件名大小写完全一致。</p>
</td></tr>

<tr class='even' name='generateCpuProfile'>
  <td><code><a href='/tsconfig/#generateCpuProfile'>--generateCpuProfile</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td><p><code>profile.cpuprofile</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在编译器运行时生成 V8 CPU Profile 用于调试。</p>
</td></tr>

<tr class='odd' name='generateTrace'>
  <td><code><a href='/tsconfig/#generateTrace'>--generateTrace</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>生成事件跟踪分析和类型列表。</p>
</td></tr>

<tr class='even' name='importHelpers'>
  <td><code><a href='/tsconfig/#importHelpers'>--importHelpers</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>允许在每个项目中从 tslib 统一导入辅助函数，而不是在每个文件中重复包含它们。</p>
</td></tr>

<tr class='odd' name='importsNotUsedAsValues'>
  <td><code><a href='/tsconfig/#importsNotUsedAsValues'>--importsNotUsedAsValues</a></code></td>
  <td><p><code>remove</code>, <code>preserve</code>, or <code>error</code></p>
</td>
  <td><p><code>remove</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定仅用于类型的导入的生成与检查行为。</p>
</td></tr>

<tr class='even' name='incremental'>
  <td><code><a href='/tsconfig/#incremental'>--incremental</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#composite"><code>composite</code></a>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>保存 .tsbuildinfo 文件以实现项目的增量编译。</p>
</td></tr>

<tr class='odd' name='inlineSourceMap'>
  <td><code><a href='/tsconfig/#inlineSourceMap'>--inlineSourceMap</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>在生成的 JavaScript 文件中内联包含 sourcemap 内容。</p>
</td></tr>

<tr class='even' name='inlineSources'>
  <td><code><a href='/tsconfig/#inlineSources'>--inlineSources</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>将源代码作为内联内容包含在生成的 JavaScript 中的 source map 内。</p>
</td></tr>

<tr class='odd' name='isolatedDeclarations'>
  <td><code><a href='/tsconfig/#isolatedDeclarations'>--isolatedDeclarations</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>要求在导出项上提供充分的类型注解，以便其他工具可以轻松生成声明文件。</p>
</td></tr>

<tr class='even' name='isolatedModules'>
  <td><code><a href='/tsconfig/#isolatedModules'>--isolatedModules</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#verbatimModuleSyntax"><code>verbatimModuleSyntax</code></a>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>确保每个文件都能安全地转译，而无需依赖其他导入。</p>
</td></tr>

<tr class='odd' name='jsx'>
  <td><code><a href='/tsconfig/#jsx'>--jsx</a></code></td>
  <td><p><code>preserve</code>, <code>react</code>, <code>react-native</code>, <code>react-jsx</code>, or <code>react-jsxdev</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定生成何种 JSX 代码。</p>
</td></tr>

<tr class='even' name='jsxFactory'>
  <td><code><a href='/tsconfig/#jsxFactory'>--jsxFactory</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td><p><code>React.createElement</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定生成 React JSX 代码时使用的 JSX 工厂函数，例如 'React.createElement' 或 'h'。</p>
</td></tr>

<tr class='odd' name='jsxFragmentFactory'>
  <td><code><a href='/tsconfig/#jsxFragmentFactory'>--jsxFragmentFactory</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td><p><code>React.Fragment</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定生成 React JSX 代码时片段所使用的 JSX 片段引用，例如 'React.Fragment' 或 'Fragment'。</p>
</td></tr>

<tr class='even' name='jsxImportSource'>
  <td><code><a href='/tsconfig/#jsxImportSource'>--jsxImportSource</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td><p><code>react</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在使用 <code>jsx: react-jsx*</code> 时，指定用于导入 JSX 工厂函数的模块标识符。</p>
</td></tr>

<tr class='odd' name='keyofStringsOnly'>
  <td><code><a href='/tsconfig/#keyofStringsOnly'>--keyofStringsOnly</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>让 keyof 仅返回 string，而非 string、number 或 symbol。已废弃的历史选项。</p>
</td></tr>

<tr class='even' name='lib'>
  <td><code><a href='/tsconfig/#lib'>--lib</a></code></td>
  <td><p><code>list</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定一组描述目标运行时环境的内置库声明文件。</p>
</td></tr>

<tr class='odd' name='libReplacement'>
  <td><code><a href='/tsconfig/#libReplacement'>--libReplacement</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>允许使用自定义库文件替换默认的 <code>lib</code> 文件。</p>
</td></tr>

<tr class='even' name='listEmittedFiles'>
  <td><code><a href='/tsconfig/#listEmittedFiles'>--listEmittedFiles</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在编译后输出生成的文件名称。</p>
</td></tr>

<tr class='odd' name='listFiles'>
  <td><code><a href='/tsconfig/#listFiles'>--listFiles</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>打印编译过程中读取的所有文件。</p>
</td></tr>

<tr class='even' name='mapRoot'>
  <td><code><a href='/tsconfig/#mapRoot'>--mapRoot</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定调试器查找 source map 文件的位置，而不是使用生成时的相对位置。</p>
</td></tr>

<tr class='odd' name='maxNodeModuleJsDepth'>
  <td><code><a href='/tsconfig/#maxNodeModuleJsDepth'>--maxNodeModuleJsDepth</a></code></td>
  <td><p><code>number</code></p>
</td>
  <td><p><code>0</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定在 <code>node_modules</code> 中检查 JavaScript 文件时所允许的最大文件夹深度。仅适用于 <a href="#allowJs"><code>allowJs</code></a>。</p>
</td></tr>

<tr class='even' name='module'>
  <td><code><a href='/tsconfig/#module'>--module</a></code></td>
  <td><p><code>none</code>, <code>commonjs</code>, <code>amd</code>, <code>umd</code>, <code>system</code>, <code>es6</code>/<code>es2015</code>, <code>es2020</code>, <code>es2022</code>, <code>esnext</code>, <code>node16</code>, <code>node18</code>, <code>node20</code>, <code>nodenext</code>, or <code>preserve</code></p>
</td>
  <td><p><code>CommonJS</code> if <a href="#target"><code>target</code></a> is <code>ES5</code>; <code>ES6</code>/<code>ES2015</code> otherwise.</p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定生成什么模块代码。</p>
</td></tr>

<tr class='odd' name='moduleDetection'>
  <td><code><a href='/tsconfig/#moduleDetection'>--moduleDetection</a></code></td>
  <td><p><code>legacy</code>, <code>auto</code>, or <code>force</code></p>
</td>
  <td><p>"auto": Treat files with imports, exports, import.meta, jsx (with jsx: react-jsx), or esm format (with module: node16+) as modules.</p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定用于检测文件是脚本还是模块的方法。</p>
</td></tr>

<tr class='even' name='moduleResolution'>
  <td><code><a href='/tsconfig/#moduleResolution'>--moduleResolution</a></code></td>
  <td><p><code>classic</code>, <code>node10</code>/<code>node</code>, <code>node16</code>, <code>nodenext</code>, or <code>bundler</code></p>
</td>
  <td><p><code>Node10</code> if <a href="#module"><code>module</code></a> is <code>CommonJS</code>; <code>Node16</code> if <a href="#module"><code>module</code></a> is <code>Node16</code>, <code>Node18</code>, or <code>Node20</code>; <code>NodeNext</code> if <a href="#module"><code>module</code></a> is <code>NodeNext</code>; <code>Bundler</code> if <a href="#module"><code>module</code></a> is <code>Preserve</code>; <code>Classic</code> otherwise.</p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定 TypeScript 如何根据给定的模块说明符查找文件。</p>
</td></tr>

<tr class='odd' name='moduleSuffixes'>
  <td><code><a href='/tsconfig/#moduleSuffixes'>--moduleSuffixes</a></code></td>
  <td><p><code>list</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>解析模块时要搜索的文件名后缀列表。</p>
</td></tr>

<tr class='even' name='newLine'>
  <td><code><a href='/tsconfig/#newLine'>--newLine</a></code></td>
  <td><p><code>crlf</code> or <code>lf</code></p>
</td>
  <td><p><code>lf</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>设置生成文件时的换行字符。</p>
</td></tr>

<tr class='odd' name='noCheck'>
  <td><code><a href='/tsconfig/#noCheck'>--noCheck</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>禁用完整的类型检查（仅报告严重的解析和输出错误）。</p>
</td></tr>

<tr class='even' name='noEmit'>
  <td><code><a href='/tsconfig/#noEmit'>--noEmit</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>禁用在编译中输出文件。</p>
</td></tr>

<tr class='odd' name='noEmitHelpers'>
  <td><code><a href='/tsconfig/#noEmitHelpers'>--noEmitHelpers</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>禁用在编译输出中生成类似 <code>__extends</code> 的自定义辅助函数。</p>
</td></tr>

<tr class='even' name='noEmitOnError'>
  <td><code><a href='/tsconfig/#noEmitOnError'>--noEmitOnError</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>报告任何类型检查错误时不生成文件。</p>
</td></tr>

<tr class='odd' name='noErrorTruncation'>
  <td><code><a href='/tsconfig/#noErrorTruncation'>--noErrorTruncation</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>禁止在错误信息中截断类型。</p>
</td></tr>

<tr class='even' name='noFallthroughCasesInSwitch'>
  <td><code><a href='/tsconfig/#noFallthroughCasesInSwitch'>--noFallthroughCasesInSwitch</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>对 switch 语句中贯穿（fallthrough）的 case 分支启用错误报告。</p>
</td></tr>

<tr class='odd' name='noImplicitAny'>
  <td><code><a href='/tsconfig/#noImplicitAny'>--noImplicitAny</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#strict"><code>strict</code></a>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>为具有隐式 <code>any</code> 类型的表达式和声明启用错误报告。</p>
</td></tr>

<tr class='even' name='noImplicitOverride'>
  <td><code><a href='/tsconfig/#noImplicitOverride'>--noImplicitOverride</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>确保派生类中重写基类成员的方法都标记有 override 修饰符。</p>
</td></tr>

<tr class='odd' name='noImplicitReturns'>
  <td><code><a href='/tsconfig/#noImplicitReturns'>--noImplicitReturns</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>对函数中未显式 return 的代码分支报错。</p>
</td></tr>

<tr class='even' name='noImplicitThis'>
  <td><code><a href='/tsconfig/#noImplicitThis'>--noImplicitThis</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#strict"><code>strict</code></a>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>当 <code>this</code> 表达式具有隐含的 <code>any</code> 类型时启用错误报告。</p>
</td></tr>

<tr class='odd' name='noImplicitUseStrict'>
  <td><code><a href='/tsconfig/#noImplicitUseStrict'>--noImplicitUseStrict</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>禁止在生成的 JavaScript 文件中添加 'use strict' 指令。</p>
</td></tr>

<tr class='even' name='noLib'>
  <td><code><a href='/tsconfig/#noLib'>--noLib</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>禁用包含任何库文件，包括默认的 lib.d.ts。</p>
</td></tr>

<tr class='odd' name='noPropertyAccessFromIndexSignature'>
  <td><code><a href='/tsconfig/#noPropertyAccessFromIndexSignature'>--noPropertyAccessFromIndexSignature</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>强制要求对使用索引类型声明的键使用索引访问器。</p>
</td></tr>

<tr class='even' name='noResolve'>
  <td><code><a href='/tsconfig/#noResolve'>--noResolve</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>禁止通过 <code>import</code>、<code>require</code> 或 <code>&#x3C;reference></code> 增加 TypeScript 应添加到项目中的文件数量。</p>
</td></tr>

<tr class='odd' name='noStrictGenericChecks'>
  <td><code><a href='/tsconfig/#noStrictGenericChecks'>--noStrictGenericChecks</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>禁用函数类型中泛型签名的严格检查。</p>
</td></tr>

<tr class='even' name='noUncheckedIndexedAccess'>
  <td><code><a href='/tsconfig/#noUncheckedIndexedAccess'>--noUncheckedIndexedAccess</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>当使用索引访问时，将 <code>undefined</code> 添加到类型中。</p>
</td></tr>

<tr class='odd' name='noUncheckedSideEffectImports'>
  <td><code><a href='/tsconfig/#noUncheckedSideEffectImports'>--noUncheckedSideEffectImports</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>检查副作用导入。</p>
</td></tr>

<tr class='even' name='noUnusedLocals'>
  <td><code><a href='/tsconfig/#noUnusedLocals'>--noUnusedLocals</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>当局部变量未被读取时启用报错。</p>
</td></tr>

<tr class='odd' name='noUnusedParameters'>
  <td><code><a href='/tsconfig/#noUnusedParameters'>--noUnusedParameters</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>当函数参数未被读取时报错。</p>
</td></tr>

<tr class='even' name='out'>
  <td><code><a href='/tsconfig/#out'>--out</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>已弃用的配置项。请改用 <a href="#outFile"><code>outFile</code></a>。</p>
</td></tr>

<tr class='odd' name='outDir'>
  <td><code><a href='/tsconfig/#outDir'>--outDir</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>为所有生成的文件指定输出目录。</p>
</td></tr>

<tr class='even' name='outFile'>
  <td><code><a href='/tsconfig/#outFile'>--outFile</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定一个将所有输出打包合并为一个 JavaScript 文件的路径。如果 <a href="#declaration"><code>declaration</code></a> 为 true，还将指定一个打包所有 .d.ts 输出的文件。</p>
</td></tr>

<tr class='odd' name='paths'>
  <td><code><a href='/tsconfig/#paths'>--paths</a></code></td>
  <td><p><code>object</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定一组将导入重映射到其他查找位置的条目。</p>
</td></tr>

<tr class='even' name='plugins'>
  <td><code><a href='/tsconfig/#plugins'>--plugins</a></code></td>
  <td><p><code>list</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定要引入的语言服务插件列表。</p>
</td></tr>

<tr class='odd' name='preserveConstEnums'>
  <td><code><a href='/tsconfig/#preserveConstEnums'>--preserveConstEnums</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#isolatedModules"><code>isolatedModules</code></a>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>禁止在生成的代码中擦除 <code>const enum</code> 声明。</p>
</td></tr>

<tr class='even' name='preserveSymlinks'>
  <td><code><a href='/tsconfig/#preserveSymlinks'>--preserveSymlinks</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>禁止将符号链接解析为其真实路径。这与 Node.js 中的同名标志相对应。</p>
</td></tr>

<tr class='odd' name='preserveValueImports'>
  <td><code><a href='/tsconfig/#preserveValueImports'>--preserveValueImports</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>在 JavaScript 输出中保留未使用的导入值（否则这些值会被移除）。</p>
</td></tr>

<tr class='even' name='preserveWatchOutput'>
  <td><code><a href='/tsconfig/#preserveWatchOutput'>--preserveWatchOutput</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在监视模式下禁止清空控制台屏幕。</p>
</td></tr>

<tr class='odd' name='pretty'>
  <td><code><a href='/tsconfig/#pretty'>--pretty</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>在 TypeScript 输出中启用颜色和格式化，使编译器错误更易于阅读。</p>
</td></tr>

<tr class='even' name='reactNamespace'>
  <td><code><a href='/tsconfig/#reactNamespace'>--reactNamespace</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td><p><code>React</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定调用 <code>createElement</code> 的对象。这仅在针对 <code>react</code> JSX 输出时适用。</p>
</td></tr>

<tr class='odd' name='removeComments'>
  <td><code><a href='/tsconfig/#removeComments'>--removeComments</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>禁止生成注释。</p>
</td></tr>

<tr class='even' name='resolveJsonModule'>
  <td><code><a href='/tsconfig/#resolveJsonModule'>--resolveJsonModule</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>允许导入 .json 文件。</p>
</td></tr>

<tr class='odd' name='resolvePackageJsonExports'>
  <td><code><a href='/tsconfig/#resolvePackageJsonExports'>--resolvePackageJsonExports</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> when <a href="#moduleResolution"><code>moduleResolution</code></a> is <code>node16</code>, <code>nodenext</code>, or <code>bundler</code>; otherwise <code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>在解析包导入时使用 package.json 的 'exports' 字段。</p>
</td></tr>

<tr class='even' name='resolvePackageJsonImports'>
  <td><code><a href='/tsconfig/#resolvePackageJsonImports'>--resolvePackageJsonImports</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> when <a href="#moduleResolution"><code>moduleResolution</code></a> is <code>node16</code>, <code>nodenext</code>, or <code>bundler</code>; otherwise <code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在解析导入时使用 package.json 的 'imports' 字段。</p>
</td></tr>

<tr class='odd' name='rewriteRelativeImportExtensions'>
  <td><code><a href='/tsconfig/#rewriteRelativeImportExtensions'>--rewriteRelativeImportExtensions</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>在输出文件中将相对导入路径中的 <code>.ts</code>、<code>.tsx</code>、<code>.mts</code> 和 <code>.cts</code> 文件扩展名重写为对应的 JavaScript 扩展名。</p>
</td></tr>

<tr class='even' name='rootDir'>
  <td><code><a href='/tsconfig/#rootDir'>--rootDir</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td><p>Computed from the list of input files.</p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定源文件中的根目录。</p>
</td></tr>

<tr class='odd' name='rootDirs'>
  <td><code><a href='/tsconfig/#rootDirs'>--rootDirs</a></code></td>
  <td><p><code>list</code></p>
</td>
  <td><p>Computed from the list of input files.</p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>允许在解析模块时将多个文件夹视为同一个。</p>
</td></tr>

<tr class='even' name='skipDefaultLibCheck'>
  <td><code><a href='/tsconfig/#skipDefaultLibCheck'>--skipDefaultLibCheck</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>跳过对 TypeScript 内置的 .d.ts 文件的类型检查。</p>
</td></tr>

<tr class='odd' name='skipLibCheck'>
  <td><code><a href='/tsconfig/#skipLibCheck'>--skipLibCheck</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>跳过对所有 .d.ts 声明文件的类型检查。</p>
</td></tr>

<tr class='even' name='sourceMap'>
  <td><code><a href='/tsconfig/#sourceMap'>--sourceMap</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>为生成的 JavaScript 文件创建 source map 文件。</p>
</td></tr>

<tr class='odd' name='sourceRoot'>
  <td><code><a href='/tsconfig/#sourceRoot'>--sourceRoot</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定调试器查找引用源文件的根路径。</p>
</td></tr>

<tr class='even' name='stableTypeOrdering'>
  <td><code><a href='/tsconfig/#stableTypeOrdering'>--stableTypeOrdering</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>确保在不同编译之间类型以稳定且确定性的方式排序。</p>
</td></tr>

<tr class='odd' name='stopBuildOnErrors'>
  <td><code><a href='/tsconfig/#stopBuildOnErrors'>--stopBuildOnErrors</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>在上游项目发生错误时跳过构建下游项目。</p>
</td></tr>

<tr class='even' name='strict'>
  <td><code><a href='/tsconfig/#strict'>--strict</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>启用所有严格类型检查选项。</p>
</td></tr>

<tr class='odd' name='strictBindCallApply'>
  <td><code><a href='/tsconfig/#strictBindCallApply'>--strictBindCallApply</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#strict"><code>strict</code></a>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>检查 <code>bind</code>、<code>call</code> 和 <code>apply</code> 方法的参数是否与原函数相匹配。</p>
</td></tr>

<tr class='even' name='strictBuiltinIteratorReturn'>
  <td><code><a href='/tsconfig/#strictBuiltinIteratorReturn'>--strictBuiltinIteratorReturn</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#strict"><code>strict</code></a>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>内置迭代器在实例化时的 TReturn 类型为 undefined 而不是 any。</p>
</td></tr>

<tr class='odd' name='strictFunctionTypes'>
  <td><code><a href='/tsconfig/#strictFunctionTypes'>--strictFunctionTypes</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#strict"><code>strict</code></a>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>在对函数赋值时，检查确保参数和返回值满足子类型兼容性。</p>
</td></tr>

<tr class='even' name='strictNullChecks'>
  <td><code><a href='/tsconfig/#strictNullChecks'>--strictNullChecks</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#strict"><code>strict</code></a>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在进行类型检查时考虑 <code>null</code> 和 <code>undefined</code>。</p>
</td></tr>

<tr class='odd' name='strictPropertyInitialization'>
  <td><code><a href='/tsconfig/#strictPropertyInitialization'>--strictPropertyInitialization</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#strict"><code>strict</code></a>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>检查类中已声明但在构造函数中未明确初始化的属性。</p>
</td></tr>

<tr class='even' name='stripInternal'>
  <td><code><a href='/tsconfig/#stripInternal'>--stripInternal</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>禁止为 JSDoc 注释中带有 <code>@internal</code> 的代码生成声明。</p>
</td></tr>

<tr class='odd' name='suppressExcessPropertyErrors'>
  <td><code><a href='/tsconfig/#suppressExcessPropertyErrors'>--suppressExcessPropertyErrors</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>禁用在创建对象字面量时报告额外属性错误。</p>
</td></tr>

<tr class='even' name='suppressImplicitAnyIndexErrors'>
  <td><code><a href='/tsconfig/#suppressImplicitAnyIndexErrors'>--suppressImplicitAnyIndexErrors</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>在对缺少索引签名的对象进行索引时，抑制 <a href="#noImplicitAny"><code>noImplicitAny</code></a> 错误。</p>
</td></tr>

<tr class='odd' name='target'>
  <td><code><a href='/tsconfig/#target'>--target</a></code></td>
  <td><p><code>es3</code>, <code>es5</code>, <code>es6</code>/<code>es2015</code>, <code>es2016</code>, <code>es2017</code>, <code>es2018</code>, <code>es2019</code>, <code>es2020</code>, <code>es2021</code>, <code>es2022</code>, <code>es2023</code>, <code>es2024</code>, <code>es2025</code>, or <code>esnext</code></p>
</td>
  <td><p><code>es2023</code> if <a href="#module"><code>module</code></a> is <code>node20</code>; <code>esnext</code> if <a href="#module"><code>module</code></a> is <code>nodenext</code>; <code>ES5</code> otherwise.</p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>设置生成的 JavaScript 语言版本，并包含兼容的库声明。</p>
</td></tr>

<tr class='even' name='traceResolution'>
  <td><code><a href='/tsconfig/#traceResolution'>--traceResolution</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>记录 <a href="#moduleResolution"><code>moduleResolution</code></a> 过程中的路径查找信息。</p>
</td></tr>

<tr class='odd' name='tsBuildInfoFile'>
  <td><code><a href='/tsconfig/#tsBuildInfoFile'>--tsBuildInfoFile</a></code></td>
  <td><p><code>string</code></p>
</td>
  <td><p><code>.tsbuildinfo</code></p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>用于存储 <code>.tsbuildinfo</code> 增量构建信息的文件。</p>
</td></tr>

<tr class='even' name='typeRoots'>
  <td><code><a href='/tsconfig/#typeRoots'>--typeRoots</a></code></td>
  <td><p><code>list</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>指定多个行为类似于 <code>./node_modules/@types</code> 的目录。</p>
</td></tr>

<tr class='odd' name='types'>
  <td><code><a href='/tsconfig/#types'>--types</a></code></td>
  <td><p><code>list</code></p>
</td>
  <td></td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>指定包含的类型声明包名称，这些包无需在源文件中被显式引用。</p>
</td></tr>

<tr class='even' name='useDefineForClassFields'>
  <td><code><a href='/tsconfig/#useDefineForClassFields'>--useDefineForClassFields</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#target"><code>target</code></a> is <code>ES2022</code> or higher, including <code>ESNext</code>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>输出符合 ECMAScript 标准规范的类字段。</p>
</td></tr>

<tr class='odd' name='useUnknownInCatchVariables'>
  <td><code><a href='/tsconfig/#useUnknownInCatchVariables'>--useUnknownInCatchVariables</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>true</code> if <a href="#strict"><code>strict</code></a>; <code>false</code> otherwise.</p>
</td>
</tr>
<tr class="option-description odd"><td colspan="3">
<p>将 catch 子句中的变量默认类型设为 <code>unknown</code> 而非 <code>any</code>。</p>
</td></tr>

<tr class='even' name='verbatimModuleSyntax'>
  <td><code><a href='/tsconfig/#verbatimModuleSyntax'>--verbatimModuleSyntax</a></code></td>
  <td><p><code>boolean</code></p>
</td>
  <td><p><code>false</code></p>
</td>
</tr>
<tr class="option-description even"><td colspan="3">
<p>不转换也不省略任何未标记为仅类型的导入或导出，确保根据 'module' 设置将其写入输出文件的格式中。</p>
</td></tr>

</tbody></table>
<!-- End of replacement  -->

## 相关内容

- 每个选项在 [TSConfig 参考](/tsconfig) 中都有完整解释。
- 了解如何使用 [`tsconfig.json`](/docs/handbook/tsconfig-json.html) 文件。
- 了解如何在 [MSBuild 项目](/docs/handbook/compiler-options-in-msbuild.html) 中工作。
