---
display: 'Synchronous Watch Directory'
oneline: '在原生不支持递归监视的平台上，同步调用回调并更新目录监视器的状态。'
---

在原生不支持递归监视的平台上，同步调用回调并更新目录监视器的状态，而不是设置一个短暂的超时以等待文件中可能发生的多次连续编辑。

```json tsconfig
{
  "watchOptions": {
    "synchronousWatchDirectory": true
  }
}
```
