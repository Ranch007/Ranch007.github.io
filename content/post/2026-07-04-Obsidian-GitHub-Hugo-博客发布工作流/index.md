---
title: "Obsidian + GitHub + Hugo 博客发布工作流"
slug: "2026-07-04-Obsidian-GitHub-Hugo-博客发布工作流"
description: ""
date: "2026-07-04T08:00:00+08:00"
image: 
math:
license:
hidden: false
draft: false
categories: ["网络技术"]
tags: [""]
---
# Obsidian + GitHub + Hugo 博客发布工作流

## 一、引言

将 Obsidian 知识库中的笔记发布到个人博客，涉及三个角色的协作：

| 角色 | 位置 | 职责 |
|------|------|------|
| **Obsidian 知识库** | 本地 `ai_wiki/` | 笔记的创作与存储 |
| **GitHub 仓库** | `Ranch007.github.io` | Hugo 博客源码 + Git 版本管理 |
| **GitHub Actions** | CI 构建 | 自动编译 Hugo 并部署到 gh-pages |

从 Obsidian 写笔记到博客上线，全自动无需手动操作博客仓库。

---

## 二、正文

### 1. 整体架构

```
┌─────────────────────┐     "发布到博客"      ┌──────────────────────┐
│   Obsidian Vault    │ ──────────────────→   │  Ranch007.github.io  │
│   ai_wiki/          │    publish-to-blog    │  Hugo 博客源码仓库   │
│                     │    .ps1 脚本执行      │                      │
│   wiki/             │                      │  content/post/       │
│   ├── it-tech/      │   1. 扫描 publish:    │  ├── xxx/index.md   │
│   ├── finance/      │      true 笔记         │  ├── yyy/index.md   │
│   └── reading/      │   2. 转换 frontmatter  │  └── ...            │
│                     │   3. 拷贝配图          │                      │
│   publish: true ────│   4. git commit & push │                      │
└─────────────────────┘                       └───────┬──────────────┘
                                                       │ push main
                                                       ▼
                                              ┌─────────────────────┐
                                              │  GitHub Actions     │
                                              │  deploy.yaml        │
                                              │                     │
                                              │  Hugo Extended      │
                                              │  v0.139.0 build     │
                                              │                     │
                                              │  → gh-pages 分支    │
                                              └─────────────────────┘
                                                       │
                                                       ▼
                                              ┌─────────────────────┐
                                              │  GitHub Pages       │
                                              │  ranch007.github.io │
                                              └─────────────────────┘
```

---

### 2. Hugo 博客站

博客仓库：[Ranch007.github.io](https://github.com/Ranch007/Ranch007.github.io)

#### 主题

使用 [CaiJimmy/hugo-theme-stack](https://github.com/CaiJimmy/hugo-theme-stack)，以 Git 子模块方式引入：

```ini
# .gitmodules
[submodule "themes/hugo-theme-stack"]
    path = themes/hugo-theme-stack
    url = https://github.com/CaiJimmy/hugo-theme-stack
```

克隆时需初始化子模块：

```bash
git clone https://github.com/Ranch007/Ranch007.github.io.git
git submodule update --init
```

#### 站点配置 (hugo.yaml) 关键点

| 配置项 | 值 | 说明 |
|--------|-----|------|
| `theme` | `hugo-theme-stack` | Stack 主题 |
| `DefaultContentLanguage` | `zh-cn` | 简体中文 |
| `hasCJKLanguage` | `true` | 中日韩语言支持，影响 `.Summary` 和 `.WordCount` |
| `baseurl` | `http://ranch007.github.io` | 站点根 URL |
| `permalinks.post` | `/p/:slug/` | 博文 URL 结构 |
| `pagination.pagerSize` | `6` | 每页文章数 |

#### 评论系统

基于 giscus（GitHub Discussions）：

```yaml
comments:
  provider: giscus
  giscus:
    repo: Ranch007/Message-Boards
    mapping: pathname
    lang: zh-CN
```

使用独立的留言板仓库 `Ranch007/Message-Boards`，与博客仓库解耦。

#### 侧边栏与头像

```yaml
sidebar:
  subtitle1: "知是行之始"
  subtitle2: "行是知之成"
  avatar:
    src: https://github.com/Ranch007.png  # GitHub 头像
```

使用了两个副标题来呈现个人格言。

#### 内容目录结构

```
content/
├── _index.md              # 首页
├── _index.zh-cn.md        # 中文首页
├── post/                  # 博文（page bundle 结构）
│   └── 文章slug/
│       └── index.md       # 博文正文 + 配图
├── page/                  # 独立页面
│   ├── about/             # 关于
│   ├── archives/          # 归档
│   ├── links/             # 友链
│   └── talk/              # 留言板
├── categories/            # 分类聚合页
│   ├── 网络技术/
│   ├── 投资理财/
│   └── 阅读随记/
└── tags/                  # 标签聚合页
```

博文使用 **Hugo page bundle**（每篇文章独立目录），方便携带封面图和资源文件。

#### 分类体系

博客侧边栏显示的三个分类，与 Obsidian wiki 的三个兴趣方向一一映射：

| Obsidian wiki 子目录 | Hugo 分类 |
|---------------------|----------|
| `wiki/it-tech/` | 网络技术 |
| `wiki/finance/` | 投资理财 |
| `wiki/reading/` | 阅读随记 |

---

### 3. 发布脚本详解

核心脚本位于 `meta/scripts/publish-to-blog.ps1`，做**一次性全量发布**而非增量同步。

#### 3.1 执行入口

**触发词**：“发布到博客”或“publish”。

**动作（定义在 `meta/CLAUDE.md` 触发 4）：**

1. AI 在笔记 frontmatter 添加 `publish: true`
2. 运行 `meta/scripts/publish-to-blog.ps1`
3. 脚本扫描 → 转换 → 复制 → git push
4. 源笔记 `publish: true` → `published: true`

#### 3.2 扫描阶段

```powershell
Get-ChildItem -Path $WikiDir -Recurse -Filter "*.md" | ForEach-Object {
    $content = Get-Content $_.FullName -Raw
    # 正则匹配 frontmatter 中的 publish: true
    if ($frontmatter -match "(?m)^publish:\s*true$") {
        $toPublish += $_
    }
}
```

递归扫描 `wiki/` 下所有 `.md` 文件，正则匹配 frontmatter 中 `publish: true` 字段。

#### 3.3 Frontmatter 转换（Obsidian → Hugo）

| Obsidian 字段 | Hugo 字段 | 转换规则 |
|-------------|----------|---------|
| `title` | `title` | 原样保留 |
| `published` | `date` | 取 `published`，无则取 `created`，均无则当天 |
| `tags` | `tags` | 原样保留，过滤掉 `publish` 标签 |
| `description` | `description` | 原样保留 |
| — | `slug` | 由标题和日期生成 |
| — | `image` | 正文第一张图片 |
| — | `categories` | 根据 wiki 子目录自动映射 |

生成的 Hugo frontmatter 示例：

```yaml
---
title: "Obsidian 个人知识库搭建全记录"
slug: "2026-07-04-Obsidian-个人知识库搭建全记录"
description: ""
date: "2026-07-04T08:00:00+08:00"
image:
math:
license:
hidden: false
draft: false
categories: ["网络技术"]
tags: [""]
---
```

#### 3.4 正文转换

**Wiki-link 转换**：所有 Obsidian 内部链接转为标准 markdown 链接：

```powershell
# [显示文字](链接.md) → [显示文字](链接.md)
$body = [regex]::Replace($body, '\[\[([^\]|]+)\|([^\]]+)\]\]', '[${2}](${1}.md)')
# [链接](链接.md) → [链接](链接.md)
$body = [regex]::Replace($body, '\[\[([^\]|]+)\]\]', '[${1}](${1}.md)')
```

**Dataview 剥离**：`dataview` 和 `dataviewjs` 代码块不兼容 Hugo，直接移除：

```powershell
$body = $body -replace '(?s)', ''
$body = $body -replace '(?s)', ''
```

**图片处理**：内嵌图片 `![assets/xxx](assets/xxx.md)` 处理逻辑：

1. 第一张图片 → **封面图**（复制到 post 根目录，写入 `image` 字段）
2. 其余图片 → 复制到 `post_slug/assets/` 子目录
3. 图片引用转为 `![](图片名)` 的相对路径
4. 如果在 assets 指定路径未找到，递归搜索 assets 所有子目录

```powershell
$body = [regex]::Replace($body, '!\[\[assets/([^\]]+)\]\]', {
    # 图片存在 → 复制 + 替换引用
    # 第一张图 -> featuredImage（封面）
    # 其余图 -> assets/ 子目录
    # 不存在 → 报错并保留原样
})
```

**空行清理**：连续三个以上换行压缩为两个。

#### 3.5 Slug 生成算法

```powershell
function Get-Slug {
    param([string]$Title, [string]$DateStr)
    # 保留中英文、数字和空格
    $clean = $Title -replace '[^\w一-鿿\s-]', ''
    # 空格替换为 -
    $clean = $clean.Trim() -replace '\s+', '-'
    if ([string]::IsNullOrWhiteSpace($clean)) { $clean = "post" }
    return "$DateStr-$clean"
}
```

最终 URL 格式：

```
https://ranch007.github.io/p/{slug-lowercase-urlencoded}/
```

例如 `2026-07-04-Obsidian 个人知识库搭建全记录` → URL 编码后变为：

```
https://ranch007.github.io/p/2026-07-04-obsidian-个人知识库搭建全记录/
```

#### 3.6 源笔记标记

发布成功后，源笔记的 `publish: true` 被替换为 `published: true`：

```powershell
$newContent = $content -replace '(?m)^publish:\s*true\s*$', 'published: true'
```

这确保了同一篇笔记不会重复发布，并在 frontmatter 中保留了发布记录。

#### 3.7 博客数据同步

脚本同步更新 vault 内的 `meta/blog-posts.md`：

```markdown
<!-- 此文件由 publish-to-blog.ps1 自动维护 -->

总篇数: 1

## 最新发布
- [文章标题](https://ranch007.github.io/p/slug/) — 2026-07-04
```

该文件被仪表盘 `index.md` 读取展示博客统计和最近发布。

#### 3.8 Git 推送

脚本自动进入博客仓库目录执行：

```powershell
Push-Location $BlogPath
git add -A
git commit -m "publish: N 篇博客更新"
git push
Pop-Location
```

推送到 `main` 分支后，GitHub Actions 会自动构建部署。

---

### 4. GitHub Actions 持续部署

工作流文件：`.github/workflows/deploy.yaml`

```yaml
name: deploy

on:
  push:
    branches:
      - main        # 只有 main 分支触发

env:
  TZ: Asia/Shanghai          # 时区设置

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
        with:
          submodules: true     # 拉取主题子模块
          fetch-depth: 0       # 完整 git 历史（用于 enableGitInfo）

      - name: Disable quotePath
        run: git config --global core.quotePath false  # 中文文件名不乱码

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "0.139.0"
          extended: true        # 必须用 extended 版（支持 SCSS）

      - name: Build
        run: hugo -D            # 构建（包括 draft 文章）

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_branch: gh-pages
          publish_dir: ./public
          commit_message: auto deploy
```

**关键细节：**

- `submodules: true` — 必须拉取 `hugo-theme-stack` 子模块，否则构建失败
- `extended: true` — Stack 主题使用 SCSS，需要 Hugo Extended 版
- `core.quotePath false` — 中文文件名在 git 中不被引用转义
- `enableGitInfo: true` — Hugo 配置中启用，利用 git 提交时间作为 `lastmod`
- 目标分支为 `gh-pages`，GitHub Pages 从该分支提供服务
- 仓库设置为 `Ranch007/Message-Boards` 作为 giscus 评论数据源

---

### 5. 完整发布流程（端到端）

**第 1 步：** 在 Obsidian 中写笔记，存入 `wiki/` 对应目录
**第 2 步：** 告诉 AI "发布到博客" 或 "把 xxx 发布到博客"
**第 3 步：** AI 在笔记 frontmatter 添加 `publish: true`（如果指定了某篇文章）
**第 4 步：** AI 执行 `meta/scripts/publish-to-blog.ps1`
**第 5 步：** 脚本扫描所有 `publish: true` 笔记：
     - 转换为 Hugo 格式
     - 复制配图
     - 写入 `content/post/{slug}/index.md`
     - 标记源笔记 `published: true`
     - 更新 `meta/blog-posts.md`
     - Git add → commit → push
**第 6 步：** GitHub Actions 检测到 push，自动构建 Hugo → 部署 gh-pages
**第 7 步：** 约 1-2 分钟后博客上线

---

### 6. 关键细节总结

#### 目录映射规则

| 源目录 | 映射为 |
|-------|-------|
| Obsidian `wiki/` | Hugo `content/post/` |
| 每个 `publish: true` 笔记 | 一个 Hugo page bundle（独立目录） |
| Obsidian `assets/` | 复制到 post bundle 内 |
| wiki 子目录（如 it-tech） | 分类标签（如"网络技术"） |

#### 前后 frontmatter 对照

**源笔记（Obsidian）：**

```yaml
---
title: Obsidian 个人知识库搭建全记录
tags: [it-tech, obsidian, tutorial, workflow]
created: 2026-07-04
publish: true
---
```

**发布后（Obsidian）：**

```yaml
---
title: Obsidian 个人知识库搭建全记录
tags: [it-tech, obsidian, tutorial, workflow]
created: 2026-07-04
published: true
---
```

**博客端（Hugo）：**

```yaml
---
title: "Obsidian 个人知识库搭建全记录"
slug: "2026-07-04-Obsidian-个人知识库搭建全记录"
date: "2026-07-04T08:00:00+08:00"
categories: ["网络技术"]
tags: [""]
draft: false
---
```

#### 本地预览博客

```bash
cd Ranch007.github.io
hugo server -D
# 访问 http://localhost:1313
```

---

## 三、拓展

### 局限性

- 目前是一次性全量扫描发布，不支持增量差异发布
- publish 脚本是 PowerShell（Windows 限定），跨平台需适配
- Wiki-link 转换后，博客端链接指向本地 `.md` 文件而非实际页面
- 如果笔记中有多条 `publish: true` 同时发布，会作为一次 git commit
- 封面图依赖 Unsplash API 自动搜索（由 AI 执行），脚本本身不包含此逻辑

### 后续可能优化

1. 支持只在笔记变更时增量复制（通过 git diff 判断）
2. 添加博客端链接重写为实际 URL 的能力
3. 支持自定义 slug 覆盖（通过 frontmatter 字段）
4. 发布前预览：在本地构建 Hugo 并预览效果再推送

---

## 四、附录

### 相关文件

- [meta/scripts/publish-to-blog.ps1](meta/scripts/publish-to-blog.ps1.md) — 发布脚本（PowerShell）
- [meta/blog-posts.md](meta/blog-posts.md.md) — 博客数据文件（自动维护）
- [meta/CLAUDE](meta/CLAUDE.md) — AI 工作规则（触发 4 定义了发布流程）

### 参考文章

- [wiki/it-tech/notes/obsidian-wiki-setup-guide](wiki/it-tech/notes/obsidian-wiki-setup-guide.md) — Obsidian 个人知识库搭建全记录（已发布到博客）
- [wiki/it-tech/notes/obsidian-ai-plugin-skill-guide](wiki/it-tech/notes/obsidian-ai-plugin-skill-guide.md) — Obsidian AI 知识库所用插件与 Skills 清单
- [CaiJimmy/hugo-theme-stack](https://github.com/CaiJimmy/hugo-theme-stack) — Stack 主题
- [peaceiris/actions-hugo](https://github.com/peaceiris/actions-hugo) — Hugo GitHub Action
- [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages) — GitHub Pages 部署 Action

### 版权信息
> 本文原载于 [Ranch's Blog](https://ranch007.github.io)，遵循 CC BY-NC-SA 4.0 协议，复制请保留原文出处。

