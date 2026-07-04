---
title: "Obsidian 个人知识库搭建全记录"
slug: "2026-07-04-Obsidian 个人知识库搭建全记录"
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
# Obsidian 个人知识库搭建全记录

## 一、引言

基于 Karpathy LLM Wiki 方法论，在 Obsidian 中搭建 AI 辅助个人知识库的完整过程。记录了每个阶段的真实决策、踩过的坑和最终方案。

---

## 二、正文

### 让 AI 阅读方法论并设计结构

> **你：** 阅读Karpathy wiki 方法论，帮我把仓库里的wiki文件夹的结构给建立起来

AI 读完后，按照 Karpathy 的三层架构（raw / wiki / schema）设计了初始目录。经过讨论，确定了三个兴趣方向：

```
it-tech/    ← IT 技术（编程、框架、工具、架构）
finance/    ← 投资理财（概念、策略、标的分析）
reading/    ← 阅读随记（书籍、文章、跨书主题）
```

初始结构采用了严格的嵌套方案，后来扁平化调整，去掉了多余层级。

### 目录演变

```
初始方案（三层嵌套）            最终方案（扁平）
─────────────────             ────────────────
ai_wiki/                      ai_wiki/
├── wiki/                     ├── raw/          ← 原始资料
│   ├── raw/                  ├── wiki/         ← 知识库
│   ├── wiki/                 │   ├── it-tech/
│   │   ├── it-tech/          │   ├── finance/
│   │   ├── finance/          │   └── reading/
│   │   └── reading/          ├── assets/       ← 配图
│   └── meta/                 └── meta/
```

### 第一篇素材

在 `raw/` 放入第一篇资料后直接说 `ingest`，AI 自动完成：归类子目录 → 读取内容 → 创建概念页 + 实体页 → 更新索引 → 写日志。

### 发现 raw/ 根目录需要整理

使用 Obsidian Web Clipper 剪藏的笔记直接掉进 `raw/` 根目录，杂乱无章。于是加了一条规则：

> 每次 Ingest 时，自动将 `raw/` 根目录下的未分类笔记按内容归类到对应子目录。

写入 `meta/CLAUDE.md`，从此再也不用手动分类。

### merge 外部 CLAUDE.md

发现 [helloianneo/obsidian-ai-second-brain](https://github.com/helloianneo/obsidian-ai-second-brain) 的 CLAUDE.md 更完整，将其与本地规则合并，形成了当前 `meta/CLAUDE.md` 的基础。

### 索引页的演变

最初的 `index.md` 只是一个简单的文章列表：

```markdown
## IT技术
- [概念页](概念页.md) — 说明
- [实体页](实体页.md) — 说明
```

用了几个来回后，逐步进化成了动态仪表盘。

### 踩过的坑

| 问题               | 原因                                                | 解决                                                          |
| ------------------ | --------------------------------------------------- | ------------------------------------------------------------- |
| 仪表盘完全不显示   | 未启用 DataviewJS 查询                              | Dataview 设置 → 开启 Enable Dataview JavaScript queries      |
| CSS 不生效         | `cssclasses` 应写作 `cssclass`（单数）          | 修正 frontmatter key                                          |
| 统计卡片对不齐     | `dv.el({container})` 参数不可靠                   | 改用`document.createElement` + `dv.container.appendChild` |
| 整个 DVJS 块不渲染 | 模板字符串内嵌 IIFE 解析失败                        | 把逻辑提到外面先算好                                          |
| 用户名读取报错     | `require('os')` 在 Obsidian 中不可用              | 改用`app.vault.adapter.basePath` 提取                       |
| 底部索引不显示     | 用`document.createElement` 但忘了 `appendChild` | 创建元素后要手动附加到容器                                    |

### 功能模块迭代顺序

1. **统计卡片** → 文章数、资料数、总计，三列等宽 Grid
2. **兴趣方向** → 各区域文章数 + 标签列表
3. **主题切换** → 🌙 emoji 按钮，一键切换深色/浅色
4. **最近更新** → 按修改时间排序，显示相对时间
5. **每日诗句** → 从诗词歌赋.md 按天轮换
6. **随机索引** → 每天随机展示各区域最多 5 篇
7. **知识图谱** → 点击跳转 Obsidian 关系图谱

### 主题切换

最初用文字按钮，后来改为纯 emoji 切换：

```javascript
btn.onclick = () => {
  const isDark = document.body.classList.contains('theme-dark');
  document.body.classList.toggle('theme-dark', !isDark);
  document.body.classList.toggle('theme-light', isDark);
  btn.textContent = isDark ? '☀️' : '🌙';
};
```

### 每日诗句

从 `wiki/reading/books/诗词歌赋.md` 读取，以当天日期为种子选一行，同一天不重复：

```javascript
const content = await dv.io.load("wiki/reading/books/诗词歌赋.md");
const poemLines = content.split('\n').filter(l => /^\s*\d+\./.test(l));
const d = Math.floor((new Date() - new Date(new Date().getFullYear(), 0, 0)) / 86400000);
const raw = poemLines[d % poemLines.length].replace(/^\s*\d+\.\s*/, '').trim();
```

### 问候语按时间段切换

| 时间段       | 问候                  |
| ------------ | --------------------- |
| 7:00–8:59   | 👋 早上好             |
| 9:00–10:59  | 👋 上午好             |
| 11:00–12:59 | 👋 中午好             |
| 13:00–16:59 | 👋 下午好             |
| 17:00–22:59 | 👋 晚上好             |
| 23:00–6:59  | 👋 夜深了，早点休息吧 |

### 用户名自动读取

从 vault 路径 `C:\Users\{userName}\...` 提取，不硬编码：

```javascript
const userName = (app.vault.adapter.basePath || '')
  .match(/[\\/]Users[\\/]([^\\/]+)/)?.[1];
```

### meta/CLAUDE.md 的演变

从一个简单规则文件逐步演变成完整的工作手册：

| 版本 | 新增内容                                                   |
| ---- | ---------------------------------------------------------- |
| v1   | Web Clipper 分类规则                                       |
| v2   | merge 外部 CLAUDE.md（Ingest / Query / Lint 三大触发行为） |
| v3   | 唤醒自动检查（哈喽/hello 触发）                            |
| v4   | 图片管理规则（命名、分类、外链处理）                       |
| v5   | Frontmatter 规范（created / published）                    |
| v6   | 精简冗余规则，保留核心流程                                 |
| v7   | log.md 改为新条目插最前面                                  |

### Frontmatter 规范

```yaml
---
title: 页面标题
tags: [主分类, 子分类]
created: 2026-07-04       # 我在 wiki 建页日期
published: 2024-06-21     # 原始资料发布日期（有时效性内容必填）
sources:
  - raw/主题/来源文件名.md
---
```

### assets/ 目录建立

```
assets/
├── it-tech/       ← IT 技术配图
├── finance/       ← 理财配图
└── reading/       ← 阅读配图
```

不按页面类型细分，一张图可能被多处引用。

### 命名规范

格式：`文章缩写-编号.扩展名`

```
xbi-analysis-01.webp
xbi-analysis-02.webp
xbi-analysis-03.webp
xbi-analysis-04.png
```

### 图片管理流程

1. Web Clipper 剪藏后按 `Ctrl+Shift+D` 下载附件到 `assets/`
2. Ingest 时检查外链图片，提示下载到本地
3. 散落的图片移入对应主题子目录
4. 引用路径统一为 `![assets/主题/xxx.png](assets/主题/xxx.png.md)`

### 初始化目录结构

> **你：** 阅读Karpathy wiki 方法论，帮我把仓库里的wiki文件夹的结构给建立起来

### 灌入第一篇资料

> **你：** ingest

### 搭建仪表盘

> **你：** 我想调整obsidian的面板，更美观，更能展现我的wiki
>
> **你：** 我希望有一个切换深色/浅色主题的按钮
>
> **你：** 仪表盘能加上每天展示一行诗句，从诗词歌赋.md读取

### 日常 Ingest

> **你：** ingest

### 健康检查

> **你：** 检查一下wiki是否搭建完善，做一轮健康检查

### 添加自定义规则

> **你：** 我使用Obsidian Web Clipper，网页笔记会直接放到raw/，你更新的时候把源笔记也在raw里面分类一下，写成一条规则到meta/CLAUDE.md

### 唤醒自动检查

> **你：** 哈喽

### 查询知识库

> **你：** 我想查询一下为什么房价会跌

---

## 三、拓展

（留空，按需补充）

---

## 四、附录

### 参考文章

- [raw/it-tech/Karpathy wiki 方法论](raw/it-tech/Karpathy wiki 方法论.md) — Karpathy LLM Wiki 方法论（原始 gist）
- [meta/CLAUDE](meta/CLAUDE.md) — AI 工作规则完整版
- [wiki/it-tech/concepts/llm-wiki-methodology](wiki/it-tech/concepts/llm-wiki-methodology.md) — LLM Wiki 方法论概念页
- [wiki/it-tech/entities/karpathy-wiki](wiki/it-tech/entities/karpathy-wiki.md) — Karpathy's LLM Wiki 实体页
- [wiki/it-tech/notes/obsidian-ai-knowledge-base-guide](wiki/it-tech/notes/obsidian-ai-knowledge-base-guide.md) — Obsidian + Claude AI 搭建指南（第三方）
- [Karpathy AI+Obsidian 知识库教程 - 栗氪聊AI](https://my.feishu.cn/wiki/LLtjwi38FiuWvKkgfPYcWiGFnab) — 飞书文档（第三方）
- [helloianneo/obsidian-ai-second-brain](https://github.com/helloianneo/obsidian-ai-second-brain) — CLAUDE.md 模板参考

### 版权信息

> 本文原载于 Ray's Wiki，请遵循 CC BY-NC-SA 4.0 协议，复制请保留原文出处。

updated: Sat Jul  4 13:30:06     2026
