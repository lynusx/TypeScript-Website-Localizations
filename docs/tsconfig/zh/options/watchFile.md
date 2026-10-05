---
display: 'Watch File'
oneline: '指定 TypeScript 监视（watch）模式的工作方式。'
---

监视单个文件所采用的策略。

- `fixedPollingInterval`：以固定时间间隔每秒多次检查每个文件的变更。
- `priorityPollingInterval`：每秒多次检查每个文件的变更，但使用启发式策略降低对某些类型文件的检查频率。
- `dynamicPriorityPolling`：使用动态队列，对较少修改的文件降低检查频率。
- `useFsEvents`（默认值）：尝试使用操作系统/文件系统的原生事件来监听文件变更。
- `useFsEventsOnParentDirectory`：尝试使用操作系统/文件系统的原生事件来监听文件所在父目录的变更。
