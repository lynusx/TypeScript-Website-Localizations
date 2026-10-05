---
display: 'Lib'
oneline: '指定一组描述目标运行时环境的内置库声明文件。'
---

TypeScript 包含了一组默认的内置 JS API 类型定义（如 `Math`），以及浏览器环境中存在的对象的类型定义（如 `document`）。
TypeScript 还会根据你指定的 [`target`](#target) 包含较新 JS 特性的 API；例如，当 [`target`](#target) 为 `ES6` 或更高版本时，`Map` 的定义就可用。

你可能会出于以下几种原因需要修改这些配置：

- 你的程序不在浏览器中运行，因此你不需要 `"dom"` 类型定义
- 你的运行时平台提供了某些 JavaScript API 对象（可能通过 polyfill 实现），但尚不支持指定 ECMAScript 版本的完整语法
- 你对更高版本 ECMAScript 中的部分（但非全部）特性拥有 polyfill 或原生实现

在 TypeScript 4.5 中，可以通过 npm 模块覆盖 lib 文件，详见[官方博客](https://devblogs.microsoft.com/typescript/announcing-typescript-4-5-beta/#supporting-lib-from-node_modules)。

### High Level libraries

| Name         | Contents                                                                                                                                          |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ES5`        | 所有 ES5 功能的核心定义                                                                                                                           |
| `ES2015`     | ES2015（亦称 ES6）中可用的附加 API —— `array.find`、`Promise`、`Proxy`、`Symbol`、`Map`、`Set`、`Reflect` 等                                    |
| `ES6`        | "ES2015" 的别名                                                                                                                                   |
| `ES2016`     | ES2016 中可用的附加 API —— `array.include` 等                                                                                                     |
| `ES7`        | "ES2016" 的别名                                                                                                                                   |
| `ES2017`     | ES2017 中可用的附加 API —— `Object.entries`、`Object.values`、`Atomics`、`SharedArrayBuffer`、`date.formatToParts`、类型化数组等                  |
| `ES2018`     | ES2018 中可用的附加 API —— `async` 可迭代对象、`promise.finally`、`Intl.PluralRules`、`regexp.groups` 等                                        |
| `ES2019`     | ES2019 中可用的附加 API —— `array.flat`、`array.flatMap`、`Object.fromEntries`、`string.trimStart`、`string.trimEnd` 等                          |
| `ES2020`     | ES2020 中可用的附加 API —— `string.matchAll` 等                                                                                                  |
| `ES2021`     | ES2021 中可用的附加 API —— `promise.any`、`string.replaceAll` 等                                                                                 |
| `ES2022`     | ES2022 中可用的附加 API —— `array.at`、`RegExp.hasIndices` 等                                                                                    |
| `ES2023`     | ES2023 中可用的附加 API —— `array.with`、`array.findLast`、`array.findLastIndex`、`array.toSorted`、`array.toReversed` 等                         |
| `ESNext`     | ESNext 中可用的附加 API —— 随着 JavaScript 规范的演进而发生变化                                                                                   |
| `DOM`        | [DOM](https://developer.mozilla.org/docs/Glossary/DOM) 定义 —— `window`、`document` 等                                                          |
| `WebWorker`  | [WebWorker](https://developer.mozilla.org/docs/Web/API/Web_Workers_API/Using_web_workers) 环境中可用的 API                                      |
| `ScriptHost` | [Windows 脚本宿主系统（Windows Script Host）](https://wikipedia.org/wiki/Windows_Script_Host) 的 API                                            |

### Individual library components

| Name                      |
| ------------------------- |
| `DOM.Iterable`            |
| `ES2015.Core`             |
| `ES2015.Collection`       |
| `ES2015.Generator`        |
| `ES2015.Iterable`         |
| `ES2015.Promise`          |
| `ES2015.Proxy`            |
| `ES2015.Reflect`          |
| `ES2015.Symbol`           |
| `ES2015.Symbol.WellKnown` |
| `ES2016.Array.Include`    |
| `ES2017.object`           |
| `ES2017.Intl`             |
| `ES2017.SharedMemory`     |
| `ES2017.String`           |
| `ES2017.TypedArrays`      |
| `ES2018.Intl`             |
| `ES2018.Promise`          |
| `ES2018.RegExp`           |
| `ES2019.Array`            |
| `ES2019.Object`           |
| `ES2019.String`           |
| `ES2019.Symbol`           |
| `ES2020.String`           |
| `ES2020.Symbol.wellknown` |
| `ES2021.Promise`          |
| `ES2021.String`           |
| `ES2021.WeakRef`          |
| `ESNext.AsyncIterable`    |
| `ESNext.Array`            |
| `ESNext.Intl`             |
| `ESNext.Symbol`           |

此列表可能未及时更新，你可以在 [TypeScript 源码](https://github.com/microsoft/TypeScript/tree/main/tsc/internal/bundled/libs)中查看完整列表。
