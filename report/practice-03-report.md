# 实践课3：Pi Agent源码安装与编译 [姓名拼音] 实验报告

**仓库地址：** ``  
**当前分支：** `main`  
**提交编号：** `957c0c49b6e957deb548a19691f7bcff57a12122`

---

## 任务一：实验准备：检查和安装开发工具

### 1. 检查现有环境
在 PowerShell 中执行了相关命令，检查开发工具版本。截图如下：
*   **Node.js:** `v24.18.0` （满足根项目 `>=22.19.0` 的要求，且符合建议使用 Node.js 24 LTS 的说明）
*   **npm:** `11.16.0` （随 Node.js 附带安装）
*   **Git:** `version 2.55.0.windows.2`


---

## 任务二：下载课程仓库源码

### 1. 下载源码
使用 `git clone` 命令下载了小组组织内 fork 的仓库。

### 2. 确认源码来源和版本
执行命令检查当前状态：
*   `Get-Location`：位于 `D:\学习\系统设计与分析\pi-hunnu-2026`
*   `git remote -v`：`origin` 指向 `git@github.com:qiqi341/pi-hunnu-2026.git`
*   `git branch --show-current`：`main`
*   `git rev-parse HEAD`：`957c0c49b6e957deb548a19691f7bcff57a12122`
*   `git status --short`：工作区干净，无未提交修改。

> **截图 S2**：显示当前目录、远端地址、分支和提交编号。  
> *(请替换为你的实际图片路径)*  
> ![检查仓库状态](images/此处填你的截图文件名2.png)

---

## 任务三：阅读配置，确认安装与编译的依据

### 1. 阅读根项目配置
打开根目录下的 `package.json`，找到以下关键配置：
*   **Node.js 版本要求**：位于 `engines` 字段中，声明为 `"node": ">=22.19.0"`。
*   **构建脚本**：位于 `scripts` 字段中的 `"build"` 命令。其内容为 `"cd packages/telemetry && npm run build && cd ../schema && npm run build && ..."`。这说明 `npm run build` 实际上是从根目录依次进入各个子包（packages），并执行子包各自的构建命令，且前一步成功（`&&`）才会执行下一步。

> **截图 S3**：显示 `engines.node` 和 `scripts.build` 的定义。  
> *(请替换为你的实际图片路径)*  
> ![查看配置文件](images/此处填你的截图文件名3.png)

---

## 任务四：安装项目依赖

### 1. 执行安装
在根目录执行 `npm install --ignore-scripts`。

**命令作用说明：**
*   该命令根据 `package.json` 和 `package-lock.json` 下载并安装依赖。
*   `--ignore-scripts` 参数跳过了安装阶段自动触发的生命周期脚本（如 `postinstall`、`prepare`），这有助于避免不必要的副作用，且不影响后续我们主动执行 `npm run build`。

**安装结果：**
*   添加了 332 个包，修改了 1 个包，审计了 356 个包（耗时 23秒）。
*   有 `prebuild-install` 和 `node-domexception` 的弃用 (deprecated) 警告，但不影响安装成功。
*   审计结果：`found 0 vulnerabilities`（未发现已知漏洞）。

> **截图 S4**：显示安装结束的输出结果。  
> *(请替换为你的实际图片路径)*  
> ![安装依赖](images/此处填你的截图文件名4.png)

---

## 任务五：本地编译与构建产物验证

### 1. 首次构建
执行 `npm run build`。构建过程走通了根目录到各个子包的命令。首次构建会获取模型目录数据（非下载大模型权重）。
构建结束的最后一段日志显示：`Built packages\coding-agent\dist\bundle (84 files, 9.4 MiB)`。

### 2. 检查并运行构建产物
执行以下命令验证构建结果：
*   `Test-Path .\packages\coding-agent\dist\bundle\cli.js`：输出 `True`，证明入口文件存在。
*   `node .\packages\coding-agent\dist\bundle\cli.js --version`：输出 `1.5.2`，证明编译产物可正常运行。

> **截图 S5**：显示构建结束、退出码 0、入口文件存在及版本号验证。  
> *(请替换为你的实际图片路径)*  
> ![验证构建产物](images/此处填你的截图文件名5.png)

---

## 任务六：启动本地源码版本并完成一次会话

### 1. 验证源码入口
在仓库根目录执行 `.\pi-test.bat --version`，输出 `1.5.2`。

### 2. 启动交互界面并完成会话
执行 `.\pi-test.bat` 启动交互界面。为了证明运行的是本地源码，我使用了该批处理脚本，而非全局的 `pi` 命令。

输入提示词：`请用一句话说明源码和编译产物的区别。只回答问题，不修改文件。`

**模型回复：** `源码是人写的、可读可改的原始文本；编译产物是工具从源码生成的、供运行环境直接执行的输出（如 dist/ 下的 JS/二进制），通常不手改。`

> **截图 S6**：显示 `.\pi-test.bat` 的启动、Pi 界面、自己的提示词和实际模型回复。  
> *(请替换为你的实际图片路径)*  
> ![源码运行会话](images/此处填你的截图文件名6.png)

### 报告说明
通过执行 `.\pi-test.bat`，该脚本内部由 Node.js 加载本地源码解析器并运行 `packages/coding-agent/src/cli.ts` 源码，而不是调用系统环境变量中的全局 `pi` 命令。

源码入口适合在开发阶段，修改了 `src` 目录下的代码后进行实时调试；`dist/bundle/cli.js` 是编译打包后的产物，适合在生产环境部署或最终用户直接运行，体积更小，运行效率更高。

---

## 遇到的困难及解决过程

**困难 1：安装依赖时存在弃用警告**
*   **现象**：执行 `npm install` 时出现了 `npm warn deprecated prebuild-install@7.1.3` 和 `node-domexception@1.0.0` 等警告。
*   **解决**：经过分析任务指导书，了解到 `deprecated` 警告仅表示该依赖包已不再维护，只要不是 `EBADENGINE` 或导致命令报错退出的错误，就不影响本次实验的依赖安装。最终发现 `found 0 vulnerabilities`，说明安装是成功的。

**困难 2：确保运行的是本地源码而非全局旧版本**
*   **现象**：之前使用 `pi` 命令，可能调起的是第一次实验安装的全局版本。
*   **解决**：查阅了根目录的 `pi-test.bat` 文件，发现它是运行本地源码的专用入口，通过 `node` 直接解析 `src` 目录下的 TypeScript 文件。因此在验证时，专门使用 `.\pi-test.bat` 来启动。由于本地源码版与已安装的 Pi 共用配置目录，登录状态得以保留，成功进行了对话。

---

**提交方式说明：**
我已将上述实验报告及截图上传至个人 Git 仓库。并在课程作业仓库按第一次实践课的方式提交了 Issue，Issue 名称为：`实践课3：Pi Agent源码安装与编译 [姓名拼音]`。
