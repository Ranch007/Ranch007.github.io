# Ranch's Blog

基于 Hugo + Stack 主题的个人技术博客，托管于 GitHub Pages。

## 目录结构

```
├── archetypes/       # 文章模板
├── assets/           # SCSS、JS、图片
│   ├── scss/         # 自定义样式
│   └── img/          # 头像等图片资源
├── content/          # 内容
│   ├── post/         # 博文（按日期分组）
│   └── page/         # 独立页面（关于、友链、留言等）
├── layouts/          # 自定义模板
├── static/           # 静态文件
├── themes/
│   └── hugo-theme-stack/  # 主题（Git 子模块）
└── hugo.yaml         # 站点配置
```

## 环境准备

1. 安装 [Hugo Extended](https://gohugo.io/installation/)（版本 ≥ 0.139.0）
2. 克隆仓库并初始化子模块：

```bash
git clone https://github.com/Ranch007/Ranch007.github.io.git
cd Ranch007.github.io
git submodule update --init
```

## 写文章

### 创建新文章

```bash
hugo new post/新文章标题/index.md
```

这会在 `content/post/新文章标题/` 下生成 `index.md`，包含预设的 Front Matter 模板。

### 编辑文章

打开 `content/post/新文章标题/index.md`，编辑内容。Front Matter 示例：

```yaml
---
title: 文章标题
description: 文章摘要
date: 2026-05-22
categories:
  - 日常折腾
tags:
  - blog
image: cover.png    # 文章封面图（放在同目录下）
---
```

文章封面图片直接放在文章目录下（如 `cover.png`），Hugo 会自动处理。

### 本地预览

```bash
hugo server -D
```

浏览器打开 `http://localhost:1313`，支持热更新。

## 部署

推送 `main` 分支即可，GitHub Actions 自动构建并部署到 `gh-pages` 分支，无需手动操作。

```bash
git add content/post/新文章标题/
git commit -m "新文章: XXX"
git push origin main
```

约 1-2 分钟后刷新 `https://ranch007.github.io` 即可看到新文章。

## 分类与标签

**分类**（4 个）：日常折腾、痴人呓语、网安笔记、网工笔记

**标签**：在 Front Matter 中自由添加，已有标签会自动聚合。
