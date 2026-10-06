---
title: ASP.NET Core
layout: docs
permalink: /zh/docs/handbook/asp-net-core.html
oneline: 在 ASP.NET Core 中使用 TypeScript
---

## 安装 ASP.NET Core 和 TypeScript

首先，如有需要请安装 [ASP.NET Core](https://dotnet.microsoft.com/apps/aspnet)。本快速入门指南需要 Visual Studio 2015 或 2017。

接下来，如果你使用的 Visual Studio 版本尚未安装最新的 TypeScript，可以[下载并安装](https://www.typescriptlang.org/index.html#download-links)。

## 创建新项目

1. 选择**文件（File）**
2. 选择**新建项目（New Project）**（Ctrl + Shift + N）
3. 在项目搜索栏中搜索 **.NET Core**
4. 选择 **ASP.NET Core Web Application** 并点击 _Next_ 按钮

![Visual Studio Project Window Screenshot](/images/tutorials/aspnet/createwebapp.png)

5. 为你的项目和解决方案命名。随后点击 _Create_ 按钮

![Visual Studio New Project Window Screenshot](/images/tutorials/aspnet/namewebapp.png)

6. 在最后一个窗口中，选择 **Empty** 模板并点击 _Create_ 按钮

![Visual Studio Web Application Screenshot](/images/tutorials/aspnet/emptytemplate.png)

运行应用程序，确认其能够正常工作。

![Edge 显示“Hello World”表示运行成功的屏幕截图](/images/tutorials/aspnet/workingsite.png)

### 配置服务器

打开**依赖项（Dependencies）> 管理 NuGet 程序包（Manage NuGet Packages）> 浏览（Browse）**。搜索并安装 `Microsoft.AspNetCore.StaticFiles` 和 `Microsoft.TypeScript.MSBuild`：

![The Visual Studio search for Nuget](/images/tutorials/aspnet/downloaddependency.png)

打开 `Startup.cs` 文件，将 `Configure` 函数修改为如下形式：

```cs
public void Configure(IApplicationBuilder app, IHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseDefaultFiles();
    app.UseStaticFiles();
}
```

你可能需要重启 Visual Studio，以使 `UseDefaultFiles` 和 `UseStaticFiles` 下方的红色波浪线消失。

## 添加 TypeScript

接下来，我们将添加一个新文件夹并将其命名为 `scripts`。

![The Path of "Add" then "New Folder" in Visual Studio from a Web Project](/images/tutorials/aspnet/newfolder.png)

![](/images/tutorials/aspnet/scripts.png)

## 添加 TypeScript 代码

右键单击 `scripts` 文件夹并选择**新建项（New Item）**。接着选择 **TypeScript 文件（TypeScript File）**，将文件命名为 `app.ts`

![A highlight of the new folder](/images/tutorials/aspnet/tsfile.png)

### 添加示例代码

在 `app.ts` 文件中添加以下代码：

```ts
function sayHello() {
  const compiler = (document.getElementById('compiler') as HTMLInputElement)
    .value
  const framework = (document.getElementById('framework') as HTMLInputElement)
    .value
  return `Hello from ${compiler} and ${framework}!`
}
```

## 配置构建流程

_配置 TypeScript 编译器_

首先我们需要告诉 TypeScript 如何进行构建。右键单击 `scripts` 并选择**新建项（New Item）**。接着选择 **TypeScript 配置文件（TypeScript Configuration File）**，并保留默认名称 `tsconfig.json`

![A screenshot showing the new file dialogue with TypeScript JSON Config selected](/images/tutorials/aspnet/tsconfig.png)

将 `tsconfig.json` 文件的内容替换为：

```json tsconfig
{
  "compilerOptions": {
    "noEmitOnError": true,
    "noImplicitAny": true,
    "sourceMap": true,
    "target": "es6"
  },
  "files": ["./app.ts"],
  "compileOnSave": true
}
```

- [`noEmitOnError`](/tsconfig#noEmitOnError) : 如果报告了任何错误，则不输出产物。
- [`noImplicitAny`](/tsconfig#noImplicitAny) : 在隐含 `any` 类型的表达式和声明上引发错误。
- [`sourceMap`](/tsconfig#sourceMap) : 生成对应的 `.map` 文件。
- [`target`](/tsconfig#target) : 指定 ECMAScript 目标版本。

注意：`"ESNext"` 针对最新支持的版本

在编写新代码时，启用 [`noImplicitAny`](/tsconfig#noImplicitAny) 是一个好习惯——这样可以确保你不会误写出未声明类型的代码。`"compileOnSave"` 则便于在运行中的 Web 应用里即时更新代码。

#### _配置 NPM_

我们需要配置 NPM，以便下载 JavaScript 依赖包。右键单击项目并选择**新建项（New Item）**。然后选择 **NPM 配置文件（NPM Configuration File）**，并使用默认名称 `package.json`。

![Screenshot of VS showing new file dialog with 'npm configuration file' selected](/images/tutorials/aspnet/packagejson.png)

在 `package.json` 文件的 `"devDependencies"` 部分中，添加 _gulp_ 和 _del_

```json tsconfig
"devDependencies": {
    "gulp": "4.0.2",
    "del": "5.1.0"
}
```

保存文件后，Visual Studio 应立即开始安装 gulp 和 del。如果没有自动安装，请右键单击 package.json，然后选择“还原程序包（Restore Packages）”。

随后，你应该会在解决方案资源管理器中看到一个 `npm` 文件夹

![Screenshot of VS showing npm folder](/images/tutorials/aspnet/npm.png)

#### _配置 gulp_

右键单击项目并选择**新建项（New Item）**。接着选择 **JavaScript 文件（JavaScript File）**，并将其命名为 `gulpfile.js`

```js
/// <binding AfterBuild='default' Clean='clean' />
/*
This file is the main entry point for defining Gulp tasks and using Gulp plugins.
Click here to learn more. http://go.microsoft.com/fwlink/?LinkId=518007
*/

var gulp = require('gulp')
var del = require('del')

var paths = {
  scripts: ['scripts/**/*.js', 'scripts/**/*.ts', 'scripts/**/*.map'],
}

gulp.task('clean', function () {
  return del(['wwwroot/scripts/**/*'])
})

gulp.task('default', function (done) {
  gulp.src(paths.scripts).pipe(gulp.dest('wwwroot/scripts'))
  done()
})
```

第一行代码指示 Visual Studio 在构建完成后运行 ‘default’ 任务。当你要求 Visual Studio 清理构建时，它还会运行 ‘clean’ 任务。

现在右键单击 `gulpfile.js`，然后选择“任务运行程序资源管理器（Task Runner Explorer）”。

![Screenshot of right clicking on the "Gulpfile.js" with 'Task Runner Explorer' selected](/images/tutorials/aspnet/taskrunner.png)

如果 ‘default’ 和 ‘clean’ 任务没有显示出来，请刷新资源管理器：

![Screenshot of task explorer with "Gulpfile.js" in it](/images/tutorials/aspnet/taskrunnerrefresh.png)

## 编写 HTML 页面

右键单击 `wwwroot` 文件夹（如果看不到该文件夹，请尝试构建项目），在其中添加一个名为 `index.html` 的新建项。为 `index.html` 使用以下代码

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <script src="scripts/app.js"></script>
    <title></title>
</head>
<body>
    <div id="message"></div>
    <div>
        Compiler: <input id="compiler" value="TypeScript" onkeyup="document.getElementById('message').innerText = sayHello()" /><br />
        Framework: <input id="framework" value="ASP.NET" onkeyup="document.getElementById('message').innerText = sayHello()" />
    </div>
</body>
</html>
```

## 测试

1. 运行项目
2. 当你在输入框中打字时，应该会看到提示信息实时出现/变化！

![A GIF of Edge showing the code you have just wrote](https://media.giphy.com/media/U3mTibRAx34DG3zhAN/giphy.gif)

## 调试

1. 在 Edge 浏览器中，按 F12 并点击“调试器（Debugger）”标签页。
2. 找到第一个 localhost 文件夹，然后展开 scripts/app.ts
3. 在带有 return 的行上设置一个断点。
4. 在输入框中打字，确认断点能够在 TypeScript 代码中命中，并且变量检查功能正常工作。

![An image showing the debugger running the code you have just wrote](/images/tutorials/aspnet/debugger.png)

恭喜，你已经成功构建了拥有 TypeScript 前端的 .NET Core 项目！
