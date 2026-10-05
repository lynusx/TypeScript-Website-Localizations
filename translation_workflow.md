# 基于 Vendor Branch 的英文上游变更追踪工作流指南

本文档介绍如何在 `TypeScript-Website-Localizations` 项目中使用 **Vendor Branch（供应商分支）** 模式，实现对上游英文原版文档更新的高效追踪与差异对比。

---

## 背景与问题

1. **双仓库设计**：英文原版位于上游主仓库 [`microsoft/TypeScript-Website`](https://github.com/microsoft/TypeScript-Website)，本仓库只存放多语言翻译（如 `zh/`、`ja/` 等）。
2. **Git 忽略英文**：本地通过 `yarn pull-en` 下载的 `docs/**/en` 被 `.gitignore` 显式忽略，Git 无法直接追踪英文历史。
3. **缺少内容级校验**：项目的 CI 和看板仅校验文件路径是否存在，英文内容修改时不会触发任何警报，中文翻译容易悄悄过时。

通过将英文文档临时锚定在 `zh/` 对应的路径并建立基准分支，我们可以借用 Git 的 `diff` 与 `3-way merge` 特性，精准捕获英文更新。

---

## 完整工作流

### 步骤 1：本地准备与拉取英文原版

确保本地安装依赖并拉取最新的英文原版：

```bash
yarn
yarn pull-en
```

> **说明**：`yarn pull-en` 会将英文原版拉取到本地的 `docs/**/en/` 目录中。

---

### 步骤 2：创建英文基准分支（Vendor Branch）

以翻译 `handbook-v2/Basics.md` 为例：

```bash
# 1. 创建并切换到基准分支（建议命名为 en-vendor 或 tracking-en）
git checkout -b en-vendor

# 2. 将需要翻译的英文文件拷贝到对应的中文路径下
mkdir -p docs/documentation/zh/handbook-v2
cp docs/documentation/en/handbook-v2/Basics.md docs/documentation/zh/handbook-v2/Basics.md

# 3. 代码格式化
npx prettier --write docs/documentation/zh/handbook-v2/Basics.md

# 4. 提交该英文快照作为初始锚点
git add docs/documentation/zh/handbook-v2/Basics.md
git commit -m "chore: snapshot upstream en for Basics.md"
```

---

### 步骤 3：基于基准分支创建翻译分支并翻译

```bash
# 1. 从 en-vendor 分支派生出你的翻译分支
git checkout -b feat-translate-basics

# 2. 在 docs/documentation/zh/handbook-v2/Basics.md 中进行中文翻译

# 3. 翻译完成并通过本地语法与格式检查
yarn lint docs/documentation/zh/handbook-v2/Basics.md
yarn validate-paths
npx prettier --write docs/documentation/zh/handbook-v2/Basics.md

# 4. 提交翻译
git commit -am "docs: translate Basics.md to Chinese"
```

---

### 步骤 4：在上游英文更新后进行追踪

当上游英文文档发生更新，并且你希望跟进时：

```bash
# 1. 重新拉取上游最新英文文档
yarn pull-en

# 2. 切换回 en-vendor 基准分支
git checkout en-vendor

# 3. 将最新英文再次拷贝覆盖到中文路径下
cp docs/documentation/en/handbook-v2/Basics.md docs/documentation/zh/handbook-v2/Basics.md

# 4. 提交最新英文快照
git commit -am "chore: update upstream en snapshot for Basics.md"
```

---

### 步骤 5：比对并同步变更（两种方式）

#### 方式 A：纯 Diff 对照审查（推荐，清晰明了）

直接在 `en-vendor` 分支查看前后两次英文快照的差异：

```bash
# 查看英文原版到底修改了哪些段落
git diff HEAD~1 HEAD -- docs/documentation/zh/handbook-v2/Basics.md
```

根据 diff 输出的结果，切回 `feat-translate-basics` 分支，精准对相应段落进行中文调整。

#### 方式 B：3-way Merge 冲突标记法（段落较长时极高效）

将更新后的 `en-vendor` 分支合并到你的翻译分支：

```bash
git checkout feat-translate-basics
git merge en-vendor
```

- **未被上游修改的段落**：Git 会自动保留你现有的中文翻译。
- **上游修改过的段落**：由于原英文被更新，Git 会在该段落生成 **Merge Conflict**：
  ```markdown
  <<<<<<< HEAD (当前翻译分支)
  这里是之前翻译好的旧版中文内容。
  =======
  Here is the newly updated English content from upstream.
  
  > > > > > > > en-vendor
  ```
- 你只需在编辑器中定位冲突标记，将新英文替换翻译为新中文，保存后提交合并即可。

---

### 步骤 6：向官方仓库提交干净的 PR

完成翻译或变更同步后，向微软官方仓库提交 Pull Request 时需注意：

1. **验证本地检查**：
   ```bash
   yarn test
   ```
2. **确认提交内容**：
   检查 PR 涉及的文件，确保**只包含真正已完成中文翻译的文件**，不要将包含纯英文临时文件的 `en-vendor` 分支提交或合入 PR。
3. 推送分支并提交 PR：
   ```bash
   git push origin feat-translate-basics
   ```

---

## 核心避坑指南 ⚠️

1. **切勿把包含“未翻译英文”的 `zh/` 文件合入 main 或提交官方 PR**：
   - 官方的 Issue 统计机器人（`update-github-issues`）仅根据文件是否存在来计算完成度。如果提交了纯英文占位，看板会将该文章误标为 `[x] 已完成`。
   - 官网部署时会直接读取 `zh/` 目录，若包含英文占位会导致中文网站直接展示英文原文。
2. **按需追踪（On-Demand）**：
   - 仅为你**正在翻译或维护**的文档在 `en-vendor` 中打快照，无需一次性拷贝全站数百个文档，保持基准分支轻量高效。
3. **保持基准分支纯净**：
   - `en-vendor` 分支中永远只存原汁原味的英文文件内容（即便路径带有 `zh/`），不要在 `en-vendor` 上修改任何翻译文字。
