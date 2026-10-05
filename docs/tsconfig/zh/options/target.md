---
display: 'Target'
oneline: '设置生成的 JavaScript 语言版本，并包含兼容的库声明。'
---

现代浏览器支持所有 ES6 特性，因此 `ES6` 是一个不错的选择。
如果代码部署在较旧的环境中，可以选择设置更低的目标版本；如果确保代码在更新的环境中运行，也可以选择更高的目标版本。

`target` 设置会改变哪些 JS 特性需要降级（downlevel）处理，哪些可以保持原样。
例如，如果 `target` 为 ES5 或更低版本，箭头函数 `() => this` 将被转换为等效的 `function` 表达式。

更改 `target` 还会改变 [`lib`](#lib) 的默认值。
你可以根据需要“混搭”设置 `target` 和 `lib`，但为了方便起见，通常只需设置 `target` 即可。

对于像 Node 这样的开发平台，根据平台类型及其版本会有推荐的 `target` 基准。你可以在 [tsconfig/bases](https://github.com/tsconfig/bases#centralized-recommendations-for-tsconfig-bases) 中找到一套由社区组织的 TSConfig，其中包含了常见平台及其版本的配置建议。

特殊的 `ESNext` 值代表当前 TypeScript 版本所支持的最高版本。
应谨慎使用此设置，因为它在不同的 TypeScript 版本之间含义并不相同，可能会降低升级时的可预测性。
