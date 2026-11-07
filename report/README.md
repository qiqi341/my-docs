# 实践项目2：个人实验报告

## 1. 基本信息
- GitHub 登录名：qiqi341
- 个人仓库地址：https://github.com/qiqi341/my-docs
- 最终网站地址：https://qiqi341.github.io/my-docs/

## 2. 实践过程与关键截图

### 2.1 仓库创建
操作与结果：登录GitHub后，点击New repository创建了名为my-docs的公开仓库，并勾选了添加 README文件。随后在本地使用git clone克隆到了本地。
[个人仓库](images/01-repository.png)

### 2.2 项目结构与 MkDocs 配置
操作与结果：在本地创建了docs/存放页面，report/ 存放报告。配置了 mkdocs.yml 文件，设定了主题为 material，并配置了四个页面的导航。创建了 requirements.txt 用于依赖管理。
[项目结构](images/02-structure.png)

### 2.3 本地构建或预览
操作与结果：使用 `python -m venv .venv` 创建虚拟环境并激活，安装依赖后运行 `python -m mkdocs serve` 在本地 8000 端口预览，检查了导航、表格和代码块显示正常。
[本地预览](images/03-local-build.png)

### 2.4 Pages 设置
操作与结果：进入仓库的 Settings -> Pages，将 Build and deployment 的 Source 修改为了 GitHub Actions，以确保后续工作流能够正常发布网站。
[Pages设置](images/04-pages-settings.png)

### 2.5 Actions 首次构建和部署
操作与结果：编写了 `.github/workflows/deploy.yml` 并推送到main分支。在Actions页面观察到工作流自动触发，build和deploy均显示绿色成功状态。
[首次部署](images/05-actions-first.png)

### 2.6 网站访问与四页导航检查
操作与结果：访问了 `https://qiqi341.github.io/my-docs/`，逐一打开了首页、关于、使用指南、API参考四个页面，确认了导航栏跳转正常，资源加载无误。
[网站效果](images/06-website.png)

### 2.7 第二次推送与自动更新验证
操作与结果：修改了 docs/index.md 增加了一句话，推送到 main 分支后，观察到了 Actions 自动触发了第二次运行并成功。强制刷新网站后看到了新内容。
- 对应提交链接：https://github.com/qiqi341/my-docs/commit/9bfced8
- 对应 Actions 运行链接：https://github.com/qiqi341/my-docs/actions/runs/42213238858
[第二次Actions](images/07-actions-second.png)
[网站更新](images/08-website-updated.png)

## 3. 问题与解决
实际现象：在执行 git commit 时，终端提示 `nothing to commit, working tree clean`。
关键日志：`Your branch is up to date with 'origin/main'. nothing to commit, working tree clean`。
原因：因为当时在编辑器里没有真正修改并保存 docs/index.md 文件，Git 认为没有变动。
解决操作：打开编辑器修改了文件并按 Ctrl+S 保存，再次执行 `git status` 发现显示 `modified: docs/index.md`，随后 `git add` 和 `git commit` 成功。

## 4. 原理理解
1. docs/ 和 site/ 各保存什么？为什么不用手工修改 site/？
   docs/ 保存的是 Markdown 源码文件（我们手写的）；site/ 保存的是 MkDocs 根据 docs/ 构建出来的 HTML/CSS 静态网站产物。我们不需要手工改 site/ 因为它是自动生成的，工作流会在构建时自动重新生成它并上传部署。
2. 本次工作流在什么情况下自动运行？
   当向 main 分支 push 代码，并且修改的文件路径包含 docs/**、mkdocs.yml、requirements.txt 或者工作流文件本身时自动运行。也可以在 Actions 页面手动触发 (workflow_dispatch)。
3. build 与 deploy 两个任务有什么区别？
   build 负责下载代码、安装依赖、生成静态网站（site/）并作为 artifact 上传；deploy 依赖于 build，在 build 成功后，负责将上传的 artifact 发布到 GitHub Pages。
4. 只修改 report/README.md 会触发部署吗？为什么？
   不会。因为 deploy.yml 的触发条件 paths 中只包含了 docs/**、mkdocs.yml 等，并没有包含 report/，所以修改报告不会触发网站构建，这是预期行为。
5. 本次方案为何无需创建 gh-pages 分支？
   因为采用了 GitHub 官方的 Pages Actions 部署方式，它直接利用 actions/upload-pages-artifact 和 actions/deploy-pages 将构建产物发布，不再需要像传统方式那样将产物推送到一个专门的 gh-pages 分支来托管。
