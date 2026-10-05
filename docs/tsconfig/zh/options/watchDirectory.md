---
display: 'Watch Directory'
oneline: '在缺乏递归文件监视功能的系统上，指定如何监视目录。'
---

在缺乏递归文件监视功能的系统下，监视整个目录树的策略。

- `fixedPollingInterval`：以固定时间间隔，每秒多次检查每个目录的变更。
- `dynamicPriorityPolling`：使用动态队列，对修改频率较低的目录降低检查频率。
- `useFsEvents`（默认值）：尝试使用操作系统/文件系统的原生事件来监听目录变更。
