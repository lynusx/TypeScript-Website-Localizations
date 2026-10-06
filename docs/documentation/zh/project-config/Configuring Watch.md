---
title: 配置监视模式（Watch）
layout: docs
permalink: /zh/docs/handbook/configuring-watch.html
oneline: 如何配置 TypeScript 的监视模式
translatable: true
---

从 TypeScript 3.8 开始，TypeScript 编译器提供了用于控制如何监视文件和目录的配置项。在此版本之前，配置需要通过环境变量完成，目前这些环境变量仍然可用。

## 背景

编译器的 `--watch` 实现依赖于 Node.js 的 `fs.watch` 和 `fs.watchFile`。这两种方法各有优缺点。

`fs.watch` 依赖文件系统事件来广播受监视文件和目录的变更。该机制的实现取决于操作系统且并不完全可靠——在许多操作系统上，它的表现可能不符合预期。此外，某些操作系统会限制可同时存在的监视器（watch）数量（例如某些 [Linux](https://man7.org/linux/man-pages/man7/inotify.7.html) 发行版）。在大型代码库中大量使用 `fs.watch` 可能会超出这些限制，从而导致异常行为。然而，由于该实现基于事件模型，其 CPU 占用相对较低。编译器通常使用 `fs.watch` 来监视目录（例如编译器配置文件中包含的源码目录，以及模块解析失败的目录等），TypeScript 借此来弥补单个文件监视器可能出现的失效。但是，该策略存在一个关键限制：Windows 和 macOS 支持递归监视目录，而 Linux 则不支持。这表明需要针对文件和目录监视引入更多策略。

`fs.watchFile` 采用轮询机制，因此会消耗 CPU 周期。然而，`fs.watchFile` 是目前订阅相关文件和目录事件最可靠的机制。在此策略下，TypeScript 编译器通常使用 `fs.watchFile` 监视源码文件、配置文件以及基于引用语句发现缺失的文件。这意味着使用 `fs.watchFile` 时 CPU 占用的增加程度，直接取决于代码库中被监视的文件数量。

## 在 `tsconfig.json` 中配置文件监视

推荐通过 `tsconfig.json` 中新增的 `watchOptions` 配置段来配置文件监视行为。下面提供了一个配置示例，关于可用配置项的详细说明请参阅后续章节。

```json tsconfig
{
  // 一些典型的编译器选项
  "compilerOptions": {
    "target": "es2020",
    "moduleResolution": "node"
    // ...
  },

  // 新增：文件/目录监视选项
  "watchOptions": {
    // 对文件和目录使用原生文件系统事件
    "watchFile": "useFsEvents",
    "watchDirectory": "useFsEvents",

    // 当文件频繁更新时，
    // 更频繁地轮询文件更新。
    "fallbackPolling": "dynamicPriority",

    // 不要合并监视通知
    "synchronousWatchDirectory": true,

    // 最后，还有两个额外的设置用于减少
    // 从这些目录中需要跟踪的文件数量
    "excludeDirectories": ["**/node_modules", "_build"],
    "excludeFiles": ["build/fileWhichChangesOften.ts"]
  }
}
```

更多详细信息，请参阅 [TypeScript 3.8 发布说明](/docs/handbook/release-notes/typescript-3-8.html#better-directory-watching-on-linux-and-watchoptions)。

## 使用环境变量 `TSC_WATCHFILE` 配置文件监视

<!-- prettier-ignore -->
选项                                           | 说明
-----------------------------------------------|----------------------------------------------------------------------
`PriorityPollingInterval`                      | 使用 `fs.watchFile`，但对源码文件、配置文件和缺失文件采用不同的轮询间隔
`DynamicPriorityPolling`                       | 使用动态队列，对频繁修改的文件以更短的间隔进行轮询，对未修改的文件以较低的频率进行轮询
`UseFsEvents`                                  | 使用 `fs.watch`。在限制活动监视器数量的操作系统上，当监视器创建失败时降级回退到 `fs.watchFile`。
`UseFsEventsWithFallbackDynamicPolling`        | 使用 `fs.watch`。在限制活动监视器数量的操作系统上，当监视器创建失败时降级回退到动态轮询队列（如 `DynamicPriorityPolling` 中所述）
`UseFsEventsOnParentDirectory`                 | 对所包含文件的 _父_ 目录使用 `fs.watch`（这是一种折衷方案，相比纯 `fs.watchFile` 具有更低的 CPU 占用，但准确性可能会有所降低）。
默认值（未指定时）                             | 如果环境变量 `TSC_NONPOLLING_WATCHER` 设置为 true，则使用 `UseFsEventsOnParentDirectory`。否则，使用 `fs.watchFile` 监视文件，并将任意文件的超时时间设为 `250ms`。

## 使用环境变量 `TSC_WATCHDIRECTORY` 配置目录监视

在原生不支持递归目录监视的平台（即 macOS 和 Windows 以外的操作系统）上，目录监视是通过使用由 `TSC_WATCHDIRECTORY` 指定的不同选项递归地为每个子目录创建目录监视器来实现的。

**注意：** 在原生支持递归目录监视的平台上，`TSC_WATCHDIRECTORY` 的值将被忽略。

<!-- prettier-ignore -->
选项                                           | 说明
-----------------------------------------------|----------------------------------------------------------------------
`RecursiveDirectoryUsingFsWatchFile`           | 使用 `fs.watchFile` 监视包含的目录及子目录。
`RecursiveDirectoryUsingDynamicPriorityPolling`| 使用动态轮询队列轮询包含的目录及子目录的变更。
默认值（未指定时）                             | 使用 `fs.watch` 监视包含的目录及子目录。
