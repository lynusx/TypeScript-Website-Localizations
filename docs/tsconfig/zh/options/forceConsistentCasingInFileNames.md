---
display: 'Force Consistent Casing In File Names'
oneline: '确保导入语句中的文件名大小写完全一致。'
---

TypeScript 遵循其所运行文件系统的大小写敏感规则。
如果部分开发者在大小写敏感的文件系统中工作，而其他开发者不在，这可能会导致问题。
如果某个文件尝试通过指定 `./FileManager.ts` 来导入 `fileManager.ts`，在大小写不敏感的文件系统中可以找到该文件，但在大小写敏感的文件系统中则找不到。

当启用此选项时，如果程序尝试引入文件时所用的大小写与磁盘上的实际大小写不一致，TypeScript 将会报错。
