---
display: 'Fallback Polling'
oneline: '指定在系统耗尽原生文件监视器时，监视器应使用的后备策略。'
---

当使用文件系统事件时，该选项指定了在系统耗尽原生文件监视器和/或不支持原生文件监视器时所采用的轮询策略。

- `fixedPollingInterval`：以固定时间间隔，每秒多次检查每个文件的变更。
- `priorityPollingInterval`：每秒多次检查每个文件的变更，但使用启发式算法降低某些特定类型文件的检查频率。
- `dynamicPriorityPolling`：使用动态队列，对修改频率较低的文件降低检查频率。
- `synchronousWatchDirectory`：对目录禁用延迟监视。当可能同时发生大量文件更改（例如运行 `npm install` 导致 `node_modules` 发生变更）时，延迟监视非常有用，但在某些不常见的配置中，你可能希望使用此标志禁用它。
