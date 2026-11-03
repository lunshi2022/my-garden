# 数字花园 · Obsidian 博客

把 Obsidian 里的 Markdown 笔记发布成一个带双链、反向链接、全文搜索和关系图的静态网站，部署在 GitHub Pages 上。

**零依赖**——构建脚本只用 Node 内置模块，不需要 `npm install`。

## 快速开始

```bash
# 本地预览（构建 + 起一个本地服务器）
npm run dev
# 浏览器打开 http://localhost:4321

# 只构建
npm run build
```

**不想敲命令？** 装了 Blog Uploader 插件后，在 Obsidian 里：

- 命令面板 → **「本地预览（构建并打开浏览器）」** —— 插件自己起服务（`node src/build.js --serve`）再打开浏览器，不用单开终端
- 用完执行 **「停止本地预览服务」**；端口被占用可在插件设置里改（默认 4321）
- 前提是本机装了 Node.js；没装也不影响发布，只是没法本地预览

## 部署到 GitHub Pages

1. 在 GitHub 新建一个**公开**仓库，比如 `garden`。
2. 推送本项目：

   ```bash
   git init
   git add -A
   git commit -m "init: 数字花园"
   git branch -M main
   git remote add origin https://github.com/<你的用户名>/garden.git
   git push -u origin main
   ```

3. 打开仓库的 **Settings → Pages → Build and deployment → Source**，选择 **GitHub Actions**（不是 "Deploy from a branch"）。
4. 完成。以后每次 `git push`，`.github/workflows/deploy.yml` 会自动构建并发布，站点地址是：

   `https://<你的用户名>.github.io/garden/`

5. 记得在 `site.config.json` 里把 `siteUrl` 填成上面的地址，RSS 链接才会是绝对的。

## 怎么写

把 `.md` 文件放进 `content/` 即可，子文件夹会变成侧边栏目录树。支持：

| 语法 | 写法 |
| --- | --- |
| 双链 | `[[笔记名]]`、`[[笔记名\|别名]]`、`[[笔记名#小标题]]` |
| 嵌入 | `![[图片.png]]`、`![[笔记名]]`（渲染为卡片）、音视频、PDF |
| 标签 | frontmatter `tags:` 或正文 `#标签`，自动生成标签页 |
| Callout | `> [!tip] 标题`，`[!tip]-` 默认折叠 |
| 其他 | 表格、任务列表、`==高亮==`、`~~删除线~~`、数学公式（自动加载 KaTeX） |

frontmatter 支持的字段：

```yaml
---
title: 自定义标题        # 默认取文件名或第一个一级标题
date: 2026-01-01        # 默认取文件修改时间
tags: [a, b]
aliases: [别名]         # 双链可以用别名指向这篇
description: 摘要       # 用于搜索与卡片
slug: custom-url        # 自定义输出路径（默认沿用目录结构）
draft: true             # true 则不发布
---
```

指向不存在的 `[[笔记]]` 会渲染成灰色虚线链接——这是待写清单，不是错误。

`content/index.md` 比较特殊：它不会出现在笔记列表里，内容会作为首页的开场白。

## 配套 Obsidian 插件：Blog Uploader

不想每次手动复制文件？仓库里的 `blog-uploader/` 是一个 Obsidian 插件，写完笔记点一下就完成：

> 复制笔记到博客仓库（保持库内目录结构）→ 复制它引用的全部附件 → 自动构建 → git 提交推送

**安装**（仅桌面版）：

1. 把 `blog-uploader/` 整个文件夹复制到你的 Obsidian 库：
   `<你的库>/.obsidian/plugins/blog-uploader/`（里面要有 `manifest.json` 和 `main.js`）
2. Obsidian 设置 → 第三方插件 → 关闭安全模式 → 启用 **Blog Uploader**
3. 打开插件设置，填入博客仓库根目录（比如 `D:\blog\obsidian-blog`）

**用法**：

- 左侧边栏的 ↑ 图标，或命令面板「上传当前笔记到博客」
- 文件列表里右键任意笔记 → 「上传到博客」
- 命令面板 → 「本地预览（构建并打开浏览器）」/「停止本地预览服务」：不用开终端就能看效果
- 「修改后再上传一次」就是更新（同名覆盖）；不想发布的笔记写 `draft: true`
- 设置里可分别开关「自动构建」和「自动推送」，只想攒一批再发就关掉推送
- 前提：机器上有 **git**（推送必需）；Node 可选——没装也能用，只是跳过本地构建，由 GitHub 云端构建

## Windows 一键部署器（原生 exe）

`deployer/` 里是一个 C# 写的部署器，编译成单个独立 exe：

```
双击 deployer\build.bat  →  生成 GardenDeployer.exe（约 200 KB，无任何运行时依赖）
```

双击 exe，填三项（部署目录、仓库名、GitHub 令牌），剩下的全自动：

> 检测并安装 Git / Obsidian（**不需要 Node**） → 复制站点模板 → 用 API 创建 GitHub 仓库并推送 → 自动开启 Pages → 装好 Obsidian 插件并预配置

只有一项需要你手动准备：**GitHub 访问令牌（PAT）**，因为 GitHub 已不允许用账号密码做 git 操作或调用建仓 API。界面上的「去创建令牌」会打开预填 `repo` + `workflow` 权限的页面，粘贴一次即可（之后存进 Windows 凭据管理器，插件推送时不用再输）。

首次拿到 exe 建议先跑 `GardenDeployer.exe --selftest` 自检。详见 [deployer/README.md](deployer/README.md)。

> 本机**不需要 Node**：站点由 GitHub Actions 在推送后构建。卸掉 Git / Obsidian 也不会影响笔记和已发布的网站
> （笔记是普通 `.md` 文件，网站托管在 GitHub 上），但卸掉 **Git 就无法再推送**。
> 具体说明和恢复步骤见 [deployer/README.md 的「卸载与恢复」](deployer/README.md#卸载与恢复)。

## 目录结构

```
├── blog-uploader/      # Obsidian 插件（一键上传当前笔记）
├── deployer/           # Windows 一键部署器（GUI）
├── content/            # 你的 Obsidian 笔记（.md 和附件）
├── src/
│   ├── build.js        # 构建入口
│   └── lib/
│       ├── markdown.js # Markdown 解析器
│       ├── obsidian.js # frontmatter / 双链 / 标签
│       └── templates.js# 页面模板
├── assets/             # 样式与前端脚本（会被复制进产物）
├── site/               # 构建产物（gitignore）
└── .github/workflows/deploy.yml
```

## 想改样式

所有颜色都集中在 `assets/style.css` 顶部的 CSS 变量里，改几个十六进制值就能换一套气质：

```css
:root {
  --bg: #fbfaf7;        /* 背景 */
  --accent: #17685a;    /* 主题色（链接、双链、按钮） */
  ...
}
```

深色模式自动跟随系统，右上角 ◐ 可手动切换并记住选择。

## 常见问题

- **推送后 Pages 没反应**：检查 Settings → Pages 的 Source 是否选了 GitHub Actions，以及 Actions 页面的运行日志。
- **链接里出现中文/百分号**：正常，文件名原样成为 URL，GitHub Pages 完整支持 UTF-8 路径。
- **数学公式不渲染**：KaTeX 从 CDN 加载，需要联网；离线时公式按原文显示。
- **想绑自定义域名**：在仓库根目录加 `CNAME` 文件（内容是域名），并在 `site.config.json` 的 `siteUrl` 里同步填写。
