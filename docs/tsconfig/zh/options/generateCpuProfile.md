---
display: 'Generate CPU Profile'
oneline: '在编译器运行时生成 V8 CPU Profile 用于调试。'
---

该选项允许你让 TypeScript 在编译器运行期间输出一个 V8 CPU Profile。该 CPU Profile 可用于深入分析构建速度缓慢的原因。

该选项只能通过命令行界面（CLI）使用：`--generateCpuProfile tsc-output.cpuprofile`。

```sh
npm run tsc --generateCpuProfile tsc-output.cpuprofile
```

此文件可在基于 Chromium 的浏览器（例如 Chrome 或 Edge 开发者工具）的 [CPU 分析器（CPU profiler）](https://developers.google.com/web/tools/chrome-devtools/rendering-tools/js-execution)面板中打开。
你可以在 [TypeScript Wiki 的性能部分](https://github.com/microsoft/TypeScript/wiki/Performance)了解更多关于分析编译器性能的内容。
