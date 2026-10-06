---
title: Gulp
layout: docs
permalink: /zh/docs/handbook/gulp.html
oneline: 结合 Gulp 使用 TypeScript
deprecated: true
---

本快速入门指南将教你如何使用 [gulp](https://gulpjs.com) 构建 TypeScript，并将 [Browserify](https://browserify.org)、[terser](https://terser.org) 或 [Watchify](https://github.com/substack/watchify) 添加到 gulp 管道中。
本指南还会介绍如何添加 [Babel](https://babeljs.io/) 功能（通过 [Babelify](https://github.com/babel/babelify)）。

我们假定你已经在使用 [Node.js](https://nodejs.org/) 和 [npm](https://www.npmjs.com/)。

## 最小化项目

首先新建一个目录。
我们暂且将其命名为 `proj`，你可以根据喜好自由更改名称。

```shell
mkdir proj
cd proj
```

首先，我们将按如下方式规划项目结构：

```
proj/
   ├─ src/
   └─ dist/
```

TypeScript 文件将存放在 `src` 目录中，经由 TypeScript 编译器处理后输出到 `dist` 目录。

让我们先搭好骨架：

```shell
mkdir src
mkdir dist
```

### 初始化项目

现在我们将该目录初始化为一个 npm 包。

```shell
npm init
```

系统会给出一系列提示问题。
除入口点（entry point）外，其余均可使用默认设置。
入口点请填写 `./dist/main.js`。
你随时可以在生成的 `package.json` 文件中修改这些配置。

### 安装依赖

现在我们可以使用 `npm install` 来安装包。
首先全局安装 `gulp-cli`（如果你使用的是 Unix 系统，可能需要在本指南中的 `npm install` 命令前加上 `sudo`）。

```shell
npm install -g gulp-cli
```

接着将 `typescript`、`gulp` 和 `gulp-typescript` 安装到项目的开发依赖中。
[Gulp-typescript](https://www.npmjs.com/package/gulp-typescript) 是一个用于 TypeScript 的 gulp 插件。

```shell
npm install --save-dev typescript gulp@4.0.0 gulp-typescript
```

### 编写一个简单示例

我们来编写一个 Hello World 程序。
在 `src` 下创建 `main.ts` 文件：

```ts
function hello(compiler: string) {
  console.log(`Hello from ${compiler}`)
}
hello('TypeScript')
```

在项目根目录 `proj` 下创建 `tsconfig.json` 文件：

```json tsconfig
{
  "files": ["src/main.ts"],
  "compilerOptions": {
    "noImplicitAny": true,
    "target": "es5"
  }
}
```

### 创建 `gulpfile.js`

在项目根目录下创建 `gulpfile.js` 文件：

```js
var gulp = require('gulp')
var ts = require('gulp-typescript')
var tsProject = ts.createProject('tsconfig.json')

gulp.task('default', function () {
  return tsProject.src().pipe(tsProject()).js.pipe(gulp.dest('dist'))
})
```

### 测试生成的应用

```shell
gulp
node dist/main.js
```

程序应该会输出 "Hello from TypeScript!"。

## 为代码添加模块支持

在介绍 Browserify 之前，我们先扩展一下代码并引入模块。
这种结构更贴近你在真实应用中使用的组织方式。

创建一个名为 `src/greet.ts` 的文件：

```ts
export function sayHello(name: string) {
  return `Hello from ${name}`
}
```

现在修改 `src/main.ts` 中的代码，从 `greet.ts` 中导入 `sayHello`：

```ts
import { sayHello } from './greet'

console.log(sayHello('TypeScript'))
```

最后，将 `src/greet.ts` 添加到 `tsconfig.json` 中：

```json tsconfig
{
  "files": ["src/main.ts", "src/greet.ts"],
  "compilerOptions": {
    "noImplicitAny": true,
    "target": "es5"
  }
}
```

运行 `gulp` 并在 Node 中进行测试，确保模块正常工作：

```shell
gulp
node dist/main.js
```

需要注意的是，尽管我们使用了 ES2015 模块语法，TypeScript 生成的却是 Node 所使用的 CommonJS 模块。
在本教程中我们将继续使用 CommonJS，不过你也可以通过在配置项对象中设置 `module` 来更改此行为。

## Browserify

现在我们将该项目从 Node 环境迁移到浏览器环境。
为此，我们需要将所有模块打包成单个 JavaScript 文件。
幸运的是，这正是 Browserify 的拿手好戏。
更棒的是，它允许我们使用 Node 的 CommonJS 模块系统，而这正是 TypeScript 默认输出的模块格式。
这意味着我们的 TypeScript 和 Node 配置几乎无需改动即可直接迁移到浏览器端。

首先安装 browserify、[tsify](https://www.npmjs.com/package/tsify) 和 vinyl-source-stream。
tsify 是一个 Browserify 插件，与 gulp-typescript 类似，它可以调用 TypeScript 编译器。
vinyl-source-stream 则可以将 Browserify 的文件输出转换回 gulp 能够识别的格式，即 [vinyl](https://github.com/gulpjs/vinyl)。

```shell
npm install --save-dev browserify tsify vinyl-source-stream
```

### 创建页面

在 `src` 中创建一个名为 `index.html` 的文件：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Hello World!</title>
  </head>
  <body>
    <p id="greeting">Loading ...</p>
    <script src="bundle.js"></script>
  </body>
</html>
```

现在修改 `main.ts` 来更新页面内容：

```ts
import { sayHello } from './greet'

function showHello(divName: string, name: string) {
  const elt = document.getElementById(divName)
  elt.innerText = sayHello(name)
}

showHello('greeting', 'TypeScript')
```

调用 `showHello` 会调用 `sayHello` 来修改段落文本。
现在将你的 gulpfile 修改为如下内容：

```js
var gulp = require('gulp')
var browserify = require('browserify')
var source = require('vinyl-source-stream')
var tsify = require('tsify')
var paths = {
  pages: ['src/*.html'],
}

gulp.task('copy-html', function () {
  return gulp.src(paths.pages).pipe(gulp.dest('dist'))
})

gulp.task(
  'default',
  gulp.series(gulp.parallel('copy-html'), function () {
    return browserify({
      basedir: '.',
      debug: true,
      entries: ['src/main.ts'],
      cache: {},
      packageCache: {},
    })
      .plugin(tsify)
      .bundle()
      .pipe(source('bundle.js'))
      .pipe(gulp.dest('dist'))
  }),
)
```

这里添加了 `copy-html` 任务，并将其设为 `default` 的前置依赖。
这意味着每次运行 `default` 时，`copy-html` 都会先执行。
我们还将 `default` 修改为通过 tsify 插件调用 Browserify，而不是使用 gulp-typescript。
方便的是，两者都允许我们向 TypeScript 编译器传递相同的配置项对象。

在调用 `bundle` 之后，我们使用 `source`（我们为 vinyl-source-stream 指定的别名）将输出的打包文件命名为 `bundle.js`。

运行 gulp，然后在浏览器中打开 `dist/index.html` 来测试页面。
你应该能在页面上看到 "Hello from TypeScript"。

请注意，我们为 Browserify 指定了 `debug: true`。
这会让 tsify 在打包后的 JavaScript 文件内生成 source map。
Source map 允许你在浏览器中直接调试原始 TypeScript 代码，而不是调试打包后的 JavaScript。
你可以打开浏览器的调试器并在 `main.ts` 中设置断点，以验证 source map 是否生效。
刷新页面时，断点应当会暂停页面执行，让你能够调试 `greet.ts`。

## Watchify、Babel 与 Terser

既然我们已经通过 Browserify 和 tsify 打包代码，就可以借助各种 Browserify 插件为构建流程增添更多功能。

- Watchify 会启动 gulp 并保持运行状态，每当你保存文件时进行增量编译。
  这让你能够在浏览器中维持“编辑-保存-刷新”的开发循环。

- Babel 是一个极其灵活的编译器，能将 ES2015 及更高版本的代码转换为 ES5 和 ES3。
  这允许你添加 TypeScript 不支持的、更广泛且定制化的转换规则。

- Terser 用于压缩代码，从而减少文件的下载时间。

### Watchify

我们首先使用 Watchify 来提供后台编译功能：

```shell
npm install --save-dev watchify fancy-log
```

现在将你的 gulpfile 修改为如下内容：

```js
var gulp = require('gulp')
var browserify = require('browserify')
var source = require('vinyl-source-stream')
var watchify = require('watchify')
var tsify = require('tsify')
var fancy_log = require('fancy-log')
var paths = {
  pages: ['src/*.html'],
}

var watchedBrowserify = watchify(
  browserify({
    basedir: '.',
    debug: true,
    entries: ['src/main.ts'],
    cache: {},
    packageCache: {},
  }).plugin(tsify),
)

gulp.task('copy-html', function () {
  return gulp.src(paths.pages).pipe(gulp.dest('dist'))
})

function bundle() {
  return watchedBrowserify
    .bundle()
    .on('error', fancy_log)
    .pipe(source('bundle.js'))
    .pipe(gulp.dest('dist'))
}

gulp.task('default', gulp.series(gulp.parallel('copy-html'), bundle))
watchedBrowserify.on('update', bundle)
watchedBrowserify.on('log', fancy_log)
```

这里主要有三处改动，需要对代码做一些重构：

1. 我们将 `browserify` 实例包装在 `watchify` 调用中，并保留其返回结果。
2. 我们调用了 `watchedBrowserify.on('update', bundle);`，这样每当有 TypeScript 文件发生变更时，Browserify 都会执行 `bundle` 函数。
3. 我们调用了 `watchedBrowserify.on('log', fancy_log);`，以便将日志输出到控制台。

结合 (1) 和 (2)，意味着我们必须将对 `browserify` 的调用移出 `default` 任务。
并且由于 Watchify 和 Gulp 都需要调用该函数，我们必须为 `default` 所用的函数命名。
通过 (3) 添加日志是可选的，但对于排查构建配置问题非常有用。

现在当你运行 Gulp 时，它应该会启动并保持运行状态。
尝试修改 `main.ts` 中 `showHello` 的代码并保存。
你应该会看到类似如下的输出：

```shell
proj$ gulp
[10:34:20] Using gulpfile ~/src/proj/gulpfile.js
[10:34:20] Starting 'copy-html'...
[10:34:20] Finished 'copy-html' after 26 ms
[10:34:20] Starting 'default'...
[10:34:21] 2824 bytes written (0.13 seconds)
[10:34:21] Finished 'default' after 1.36 s
[10:35:22] 2261 bytes written (0.02 seconds)
[10:35:24] 2808 bytes written (0.05 seconds)
```

### Terser

首先安装 Terser。
因为 Terser 的目的是混淆压缩代码，所以我们还需要安装 vinyl-buffer 和 gulp-sourcemaps 以保证 sourcemap 正常工作。

```shell
npm install --save-dev gulp-terser vinyl-buffer gulp-sourcemaps
```

现在将你的 gulpfile 修改为如下内容：

```js
var gulp = require('gulp')
var browserify = require('browserify')
var source = require('vinyl-source-stream')
var terser = require('gulp-terser')
var tsify = require('tsify')
var sourcemaps = require('gulp-sourcemaps')
var buffer = require('vinyl-buffer')
var paths = {
  pages: ['src/*.html'],
}

gulp.task('copy-html', function () {
  return gulp.src(paths.pages).pipe(gulp.dest('dist'))
})

gulp.task(
  'default',
  gulp.series(gulp.parallel('copy-html'), function () {
    return browserify({
      basedir: '.',
      debug: true,
      entries: ['src/main.ts'],
      cache: {},
      packageCache: {},
    })
      .plugin(tsify)
      .bundle()
      .pipe(source('bundle.js'))
      .pipe(buffer())
      .pipe(sourcemaps.init({ loadMaps: true }))
      .pipe(terser())
      .pipe(sourcemaps.write('./'))
      .pipe(gulp.dest('dist'))
  }),
)
```

请注意，`terser` 本身只需要调用一次 &mdash; 对 `buffer` 和 `sourcemaps` 的调用则是为了确保 sourcemap 继续正常工作。
这些调用使我们能够生成独立的 sourcemap 文件，而不是像之前那样使用内联 sourcemap。
现在你可以运行 Gulp，并检查 `bundle.js` 是否确实被压缩成了一团难以阅读的代码：

```shell
gulp
cat dist/bundle.js
```

### Babel

首先安装 Babelify 以及适用于 ES2015 的 Babel 预设。
与 Terser 类似，Babelify 也会改变代码结构，因此我们需要 vinyl-buffer 和 gulp-sourcemaps。
默认情况下，Babelify 只会处理扩展名为 `.js`、`.es`、`.es6` 和 `.jsx` 的文件，因此我们需要将 `.ts` 扩展名作为配置项添加到 Babelify 中。

```shell
npm install --save-dev babelify@8 babel-core babel-preset-es2015 vinyl-buffer gulp-sourcemaps
```

现在将你的 gulpfile 修改为如下内容：

```js
var gulp = require('gulp')
var browserify = require('browserify')
var source = require('vinyl-source-stream')
var tsify = require('tsify')
var sourcemaps = require('gulp-sourcemaps')
var buffer = require('vinyl-buffer')
var paths = {
  pages: ['src/*.html'],
}

gulp.task('copy-html', function () {
  return gulp.src(paths.pages).pipe(gulp.dest('dist'))
})

gulp.task(
  'default',
  gulp.series(gulp.parallel('copy-html'), function () {
    return browserify({
      basedir: '.',
      debug: true,
      entries: ['src/main.ts'],
      cache: {},
      packageCache: {},
    })
      .plugin(tsify)
      .transform('babelify', {
        presets: ['es2015'],
        extensions: ['.ts'],
      })
      .bundle()
      .pipe(source('bundle.js'))
      .pipe(buffer())
      .pipe(sourcemaps.init({ loadMaps: true }))
      .pipe(sourcemaps.write('./'))
      .pipe(gulp.dest('dist'))
  }),
)
```

我们还需要让 TypeScript 将编译目标设置为 ES2015。
这样 Babel 就能将 TypeScript 输出的 ES2015 代码转换为 ES5。
我们来修改 `tsconfig.json`：

```json tsconfig
{
  "files": ["src/main.ts"],
  "compilerOptions": {
    "noImplicitAny": true,
    "target": "es2015"
  }
}
```

对于如此简单的脚本，Babel 输出的 ES5 代码应该与 TypeScript 的直接输出非常相似。
