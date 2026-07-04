# Categories 重构设计

## 目标

将博客现有 4 个分类重构为 3 个分类，删除 2 篇过时文章。

## 变更详情

### 删除文章（2 篇）

`content/post/2024-11-10/` 下两篇网安笔记全部删除：

- Apache ActiveMQ Jolokia 后台远程代码执行漏洞（CVE-2022-41678）
- Apache ActiveMQ OpenWire 协议反序列化 RCE（CVE-2023-46604）

删除后 `2024-11-10` 目录变空，一并清理。

### 分类映射

| 旧分类 | 新分类 | 文章迁移数 |
|--------|--------|-----------|
| 日常折腾 | 网络技术 | 5 |
| 网安笔记 | 网络技术 | 4（删除 2 篇后剩余） |
| 网工笔记 | 网络技术 | 3 |
| 痴人呓语 | 阅读随记 | 1 |

新建「投资理财」分类（暂无文章，为未来准备）。

### 分类目录操作

**删除：**
- `content/categories/日常折腾/`
- `content/categories/网安笔记/`
- `content/categories/网工笔记/`
- `content/categories/痴人呓语/`

**新建：**
- `content/categories/网络技术/` — 复用「日常折腾」的 cover.png
- `content/categories/阅读随记/` — 复用「痴人呓语」的 cover.png
- `content/categories/投资理财/` — 复用「网安笔记」的 cover.png

### 配色

所有新分类沿用 `background: "#2a9d8f"` / `color: "#fff"`。

### 排序

- 网络技术 weight: 1
- 阅读随记 weight: 2
- 投资理财 weight: 3

## 影响范围

- 13 篇文章的 `categories` frontmatter
- 3 个新 `_index.md` + 3 个 `cover.png`
- 删除 4 个旧分类目录、2 篇旧文章目录及所有 assets
- 可能清理空的 `2024-11-10` 日期目录

## 不涉及

- Tags 体系不变
- 主题/布局无改动
- 文章内容不变（仅 frontmatter）
- 封面图片使用已有文件，无需新资源
