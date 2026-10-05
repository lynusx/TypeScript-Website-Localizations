---
display: 'Allow Umd Global Access'
oneline: '允许在模块内部访问 UMD 全局变量。'
---

当设置为 true 时，`allowUmdGlobalAccess` 允许你从模块文件内部将 UMD 导出项作为全局变量进行访问。模块文件是指包含 import 和/或 export 的文件。若不开启此标志，使用 UMD 模块中的导出必须通过 import 声明。

该标志的一个典型使用场景是 Web 项目：你明确知道某个特定库（例如 jQuery 或 Lodash）在运行时始终可用，但无法通过 import 来访问它。
