---
title: 与构建工具集成
layout: docs
permalink: /zh/docs/handbook/integrating-with-build-tools.html
oneline: 如何在其他构建工具中使用 TypeScript
---

## Babel

### 安装

```sh
npm install @babel/cli @babel/core @babel/preset-typescript --save-dev
```

### .babelrc

```js
{
  "presets": ["@babel/preset-typescript"]
}
```

### 使用命令行接口

```sh
./node_modules/.bin/babel --out-file bundle.js src/index.ts
```

### package.json

```js
{
  "scripts": {
    "build": "babel --out-file bundle.js main.ts"
  },
}
```

### 从命令行执行 Babel

```sh
npm run build
```

## Browserify

### 安装

```sh
npm install tsify
```

### 使用命令行接口

```sh
browserify main.ts -p [ tsify --noImplicitAny ] > bundle.js
```

### 使用 API

```js
var browserify = require('browserify')
var tsify = require('tsify')

browserify()
  .add('main.ts')
  .plugin('tsify', { noImplicitAny: true })
  .bundle()
  .pipe(process.stdout)
```

更多详情：[smrq/tsify](https://github.com/smrq/tsify)

## Grunt

### 使用 `grunt-ts`（已不再维护）

#### 安装

```sh
npm install grunt-ts --save-dev
```

#### 基础 Gruntfile.js

```js
module.exports = function (grunt) {
  grunt.initConfig({
    ts: {
      default: {
        src: ['**/*.ts', '!node_modules/**/*.ts'],
      },
    },
  })
  grunt.loadNpmTasks('grunt-ts')
  grunt.registerTask('default', ['ts'])
}
```

更多详情：[TypeStrong/grunt-ts](https://github.com/TypeStrong/grunt-ts)

### 结合使用 `grunt-browserify` 与 `tsify`

#### 安装

```sh
npm install grunt-browserify tsify --save-dev
```

#### 基础 Gruntfile.js

```js
module.exports = function (grunt) {
  grunt.initConfig({
    browserify: {
      all: {
        src: 'src/main.ts',
        dest: 'dist/main.js',
        options: {
          plugin: ['tsify'],
        },
      },
    },
  })
  grunt.loadNpmTasks('grunt-browserify')
  grunt.registerTask('default', ['browserify'])
}
```

更多详情：[jmreidy/grunt-browserify](https://github.com/jmreidy/grunt-browserify)、[TypeStrong/tsify](https://github.com/TypeStrong/tsify)

## Gulp

### 安装

```sh
npm install gulp-typescript
```

### 基础 gulpfile.js

```js
var gulp = require('gulp')
var ts = require('gulp-typescript')

gulp.task('default', function () {
  var tsResult = gulp.src('src/*.ts').pipe(
    ts({
      noImplicitAny: true,
      out: 'output.js',
    }),
  )
  return tsResult.js.pipe(gulp.dest('built/local'))
})
```

更多详情：[ivogabe/gulp-typescript](https://github.com/ivogabe/gulp-typescript)

## Jspm

### 安装

```sh
npm install -g jspm@beta
```

_注：目前 jspm 对 TypeScript 的支持处于 0.16beta 阶段_

更多详情：[TypeScriptSamples/jspm](https://github.com/Microsoft/TypeScriptSamples/tree/master/jspm)

## MSBuild

更新项目文件，在顶部引入本地安装的 `Microsoft.TypeScript.Default.props` 文件，并在底部引入 `Microsoft.TypeScript.targets` 文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <!-- Include default props at the top -->
  <Import
      Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.Default.props"
      Condition="Exists('$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.Default.props')" />

  <!-- TypeScript configurations go here -->
  <PropertyGroup Condition="'$(Configuration)' == 'Debug'">
    <TypeScriptRemoveComments>false</TypeScriptRemoveComments>
    <TypeScriptSourceMap>true</TypeScriptSourceMap>
  </PropertyGroup>
  <PropertyGroup Condition="'$(Configuration)' == 'Release'">
    <TypeScriptRemoveComments>true</TypeScriptRemoveComments>
    <TypeScriptSourceMap>false</TypeScriptSourceMap>
  </PropertyGroup>

  <!-- Include default targets at the bottom -->
  <Import
      Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets"
      Condition="Exists('$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets')" />
</Project>
```

有关定义 MSBuild 编译器选项的更多详情：[在 MSBuild 项目中设置编译器选项](/docs/handbook/compiler-options-in-msbuild.html)

## NuGet

- 右键点击 -> 管理 NuGet 程序包（Manage NuGet Packages）
- 搜索 `Microsoft.TypeScript.MSBuild`
- 点击“安装”（Install）
- 安装完成后，重新构建！

更多详情请参阅[程序包管理器对话框](http://docs.nuget.org/Consume/Package-Manager-Dialog)以及[在 NuGet 中使用每日构建版本](https://github.com/Microsoft/TypeScript/wiki/Nightly-drops#using-nuget-with-msbuild)。

## Rollup

### 安装

```
npm install @rollup/plugin-typescript --save-dev
```

注意，`typescript` 和 `tslib` 都是此插件的对等依赖项（peer dependencies），需要单独安装。

### 用法

创建一个 `rollup.config.js` [配置文件](https://www.rollupjs.org/guide/en/#configuration-files)并导入该插件：

```js
// rollup.config.js
import typescript from '@rollup/plugin-typescript'

export default {
  input: 'src/index.ts',
  output: {
    dir: 'output',
    format: 'cjs',
  },
  plugins: [typescript()],
}
```

## Svelte 编译器

### 安装

```
npm install --save-dev svelte-preprocess
```

注意，`typescript` 是此插件的可选对等依赖项（peer dependency），需要单独安装。此外，该插件也不提供 `tslib`。

你也可以考虑使用 [`svelte-check`](https://www.npmjs.com/package/svelte-check) 进行命令行类型检查。

### 用法

创建一个 `svelte.config.js` 配置文件并导入该插件：

```js
// svelte.config.js
import preprocess from 'svelte-preprocess'

const config = {
  // 请参阅 https://github.com/sveltejs/svelte-preprocess
  // 了解有关预处理器的更多信息
  preprocess: preprocess(),
}

export default config
```

现在你可以指定脚本块使用 TypeScript 编写：

```
<script lang="ts">
```

## Vite

Vite 开箱即用支持导入 `.ts` 文件。它仅执行转译而不进行类型检查。此外，它还要求某些 `compilerOptions` 具有特定取值。更多详情请参阅 [Vite 文档](https://vitejs.dev/guide/features.html#typescript)。

## Webpack

### 安装

```sh
npm install ts-loader --save-dev
```

### 使用 Webpack 5 或 4 时的基础 webpack.config.js

```js
const path = require('path')

module.exports = {
  entry: './src/index.ts',
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/,
      },
    ],
  },
  resolve: {
    extensions: ['.tsx', '.ts', '.js'],
  },
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist'),
  },
}
```

请参阅[此处关于 ts-loader 的更多详情](https://www.npmjs.com/package/ts-loader)。

替代方案：

- [awesome-typescript-loader](https://www.npmjs.com/package/awesome-typescript-loader)
