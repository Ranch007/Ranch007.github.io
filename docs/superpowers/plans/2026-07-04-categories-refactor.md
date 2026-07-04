# Categories 重构 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将博客 4 个旧分类重构为 3 个新分类（网络技术、阅读随记、投资理财），删除 2 篇过时文章，迁移 13 篇文章的 frontmatter。

**Architecture:** 纯文件系统操作——删除旧分类目录和过时文章，创建新分类 `_index.md` 并复用旧封面图，用 PowerShell 批量替换文章 frontmatter 中的 `categories` 值。

**Tech Stack:** Hugo 静态站点 + Git + PowerShell

---

### Task 1: 创建功能分支

**Files:**
- 不涉及文件修改

- [ ] **Step 1: 从 main 创建新分支**

```powershell
git checkout -b refactor/categories
```

- [ ] **Step 2: 验证分支已切换**

```powershell
git branch --show-current
```

Expected: `refactor/categories`

- [ ] **Step 3: 提交**

不提交，分支创建后开始 Task 2。

---

### Task 2: 删除 2024-11-10 两篇网安笔记文章

**Files:**
- Delete: `content/post/2024-11-10/` 下两个子目录

- [ ] **Step 1: 确认要删除的目录存在**

```powershell
Get-ChildItem -Directory "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-11-10"
```

Expected: 两个 Apache ActiveMQ 目录。

- [ ] **Step 2: 删除两篇文章目录**

```powershell
Remove-Item -Recurse -Force "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-11-10\Apache ActiveMQ Jolokia 后台远程代码执行漏洞（CVE-2022-41678）"
Remove-Item -Recurse -Force "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-11-10\Apache ActiveMQ OpenWire 协议反序列化 RCE（CVE-2023-46604）"
```

- [ ] **Step 3: 检查 2024-11-10 是否变空，是则删除空目录**

```powershell
$dir = "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-11-10"
if ((Get-ChildItem $dir -ErrorAction SilentlyContinue | Measure-Object).Count -eq 0) {
    Remove-Item $dir
}
```

- [ ] **Step 4: 验证删除结果**

```powershell
Test-Path "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-11-10"
```

Expected: `False`（目录已不存在）。

- [ ] **Step 5: 提交**

```bash
git add -A
git commit -m "feat: 删除 2024-11-10 两篇 ActiveMQ 网安笔记文章

- 删除 Apache ActiveMQ Jolokia (CVE-2022-41678)
- 删除 Apache ActiveMQ OpenWire (CVE-2023-46604)
- 清理空的 2024-11-10 日期目录

Co-Authored-By: Claude <noreply@anthropic.com>"
```

---

### Task 3: 复制旧分类封面图作为备份、删除旧分类目录

**Files:**
- Delete: `content/categories/日常折腾/`
- Delete: `content/categories/网安笔记/`
- Delete: `content/categories/网工笔记/`
- Delete: `content/categories/痴人呓语/`

- [ ] **Step 1: 先把要复用的 cover.png 临时备份出来**

```powershell
$tmpDir = "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\categories\_tmp_"
New-Item -ItemType Directory -Force -Path $tmpDir
Copy-Item "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\categories\日常折腾\cover.png" "$tmpDir\cover-折腾.png"
Copy-Item "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\categories\网安笔记\cover.png" "$tmpDir\cover-网安.png"
Copy-Item "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\categories\痴人呓语\cover.png" "$tmpDir\cover-痴人.png"
```

- [ ] **Step 2: 删除四个旧分类目录**

```powershell
Remove-Item -Recurse -Force "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\categories\日常折腾"
Remove-Item -Recurse -Force "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\categories\网安笔记"
Remove-Item -Recurse -Force "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\categories\网工笔记"
Remove-Item -Recurse -Force "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\categories\痴人呓语"
```

- [ ] **Step 3: 验证旧分类目录已全部删除**

```powershell
$olds = @("日常折腾", "网安笔记", "网工笔记", "痴人呓语")
foreach ($o in $olds) {
    $p = "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\categories\$o"
    if (Test-Path $p) { Write-Host "STILL EXISTS: $p" } else { Write-Host "DELETED: $p" }
}
```

Expected: 全部显示 `DELETED`。

- [ ] **Step 4: 提交**

```bash
git add -A
git commit -m "feat: 删除旧分类目录（日常折腾/网安笔记/网工笔记/痴人呓语）

Co-Authored-By: Claude <noreply@anthropic.com>"
```

---

### Task 4: 创建三个新分类目录（_index.md + cover.png）

**Files:**
- Create: `content/categories/网络技术/_index.md`
- Create: `content/categories/阅读随记/_index.md`
- Create: `content/categories/投资理财/_index.md`
- Copy: `content/categories/网络技术/cover.png` (来自日常折腾)
- Copy: `content/categories/阅读随记/cover.png` (来自痴人呓语)
- Copy: `content/categories/投资理财/cover.png` (来自网安笔记)

- [ ] **Step 1: 创建目录并复制封面图**

```powershell
$base = "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\categories"
$tmpDir = "$base\_tmp_"

# 网络技术 ← 日常折腾
New-Item -ItemType Directory -Force -Path "$base\网络技术"
Copy-Item "$tmpDir\cover-折腾.png" "$base\网络技术\cover.png"

# 阅读随记 ← 痴人呓语
New-Item -ItemType Directory -Force -Path "$base\阅读随记"
Copy-Item "$tmpDir\cover-痴人.png" "$base\阅读随记\cover.png"

# 投资理财 ← 网安笔记
New-Item -ItemType Directory -Force -Path "$base\投资理财"
Copy-Item "$tmpDir\cover-网安.png" "$base\投资理财\cover.png"

# 清理临时目录
Remove-Item -Recurse -Force $tmpDir
```

- [ ] **Step 2: 创建 网络技术/_index.md**

Write file `c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\categories\网络技术\_index.md`:

```markdown
---
title: "网络技术"
slug: "网络技术"
weight: 1
description: 
image: cover.png
style:
 background: "#2a9d8f"
 color: "#fff"

---
```

- [ ] **Step 3: 创建 阅读随记/_index.md**

Write file `c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\categories\阅读随记\_index.md`:

```markdown
---
title: "阅读随记"
slug: "阅读随记"
weight: 2
description: 
image: cover.png
style:
 background: "#2a9d8f"
 color: "#fff"

---
```

- [ ] **Step 4: 创建 投资理财/_index.md**

Write file `c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\categories\投资理财\_index.md`:

```markdown
---
title: "投资理财"
slug: "投资理财"
weight: 3
description: 
image: cover.png
style:
 background: "#2a9d8f"
 color: "#fff"

---
```

- [ ] **Step 5: 验证新分类目录结构**

```powershell
$news = @("网络技术", "阅读随记", "投资理财")
foreach ($n in $news) {
    $p = "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\categories\$n"
    Get-ChildItem $p | Select-Object Name
}
```

Expected: 每个目录下都有 `_index.md` 和 `cover.png`。

- [ ] **Step 6: 提交**

```bash
git add -A
git commit -m "feat: 创建新分类目录（网络技术/阅读随记/投资理财）

- 网络技术: weight 1, 封面来自日常折腾
- 阅读随记: weight 2, 封面来自痴人呓语
- 投资理财: weight 3, 封面来自网安笔记

Co-Authored-By: Claude <noreply@anthropic.com>"
```

---

### Task 5: 迁移文章 frontmatter — categories 字段

**Files:**
- Modify: 13 个文章的 `index.md` 中 `categories` 字段

- [ ] **Step 1: 日常折腾 → 网络技术（5 篇）**

```powershell
$files = @(
    "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2026-05-28\PowerShell-PROFILE-实用手册\index.md",
    "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-11-09\【转载】Windows 下包管理器 Scoop 的安装与使用\index.md",
    "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-11-09\VS Code 配置 C & C++ 编程运行环境\index.md",
    "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-10-20\【博客装修日记】Hugo + Stack + Giscus魔改美化 细节满满\index.md",
    "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-10-18\【Free】使用Hugo搭建GitHub私人博客\index.md"
)

foreach ($f in $files) {
    $content = Get-Content $f -Raw
    $content = $content -replace 'categories:\s*\["日常折腾"\]', 'categories: ["网络技术"]'
    Set-Content $f -Value $content -NoNewline
}
```

- [ ] **Step 2: 网安笔记 → 网络技术（4 篇）**

```powershell
$files = @(
    "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-11-19\APP抓包之 Burpsuite+MuMu模拟器12抓包\index.md",
    "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-11-08\掌握 Docker魔法：Windows 11 平台上的完美容器部署终极指南\index.md",
    "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-11-02\超详细的wsl2教程：windows上的linux子系统\index.md",
    "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-10-22\Win11使用JEnv管理多版本jdk\index.md"
)

foreach ($f in $files) {
    $content = Get-Content $f -Raw
    $content = $content -replace 'categories:\s*\["网安笔记"\]', 'categories: ["网络技术"]'
    Set-Content $f -Value $content -NoNewline
}
```

- [ ] **Step 3: 网工笔记 → 网络技术（3 篇）**

```powershell
$files = @(
    "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-11-12\GNS3懒人版-2.2.45安装部署详细教程\index.md",
    "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-11-11\【HCIA 路由与交换 Wakin 谢Sir】TCP-IP网络模型\index.md",
    "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-11-11\【HCIA 路由与交换 Wakin 谢Sir】数通网络基础\index.md"
)

foreach ($f in $files) {
    $content = Get-Content $f -Raw
    $content = $content -replace 'categories:\s*\["网工笔记"\]', 'categories: ["网络技术"]'
    Set-Content $f -Value $content -NoNewline
}
```

- [ ] **Step 4: 痴人呓语 → 阅读随记（1 篇）**

```powershell
$f = "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post\2024-11-11\有种一语惊醒梦中人的宿命感\index.md"
$content = Get-Content $f -Raw
$content = $content -replace 'categories:\s*\["痴人呓语"\]', 'categories: ["阅读随记"]'
Set-Content $f -Value $content -NoNewline
```

- [ ] **Step 5: 验证所有迁移：扫描全站确认无旧分类残留**

```powershell
$olds = @('日常折腾', '网安笔记', '网工笔记', '痴人呓语')
$pattern = ($olds | ForEach-Object { [regex]::Escape($_) }) -join '|'
$results = Get-ChildItem "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post" -Recurse -Filter "index.md" |
    Select-String -Pattern $pattern
if ($results) {
    Write-Host "WARNING: 旧分类残留!"
    $results | ForEach-Object { Write-Host "$($_.Path): $($_.Line)" }
} else {
    Write-Host "OK: 无旧分类残留"
}
```

Expected: `OK: 无旧分类残留`。

- [ ] **Step 6: 验证新分类出现次数正确**

```powershell
Write-Host "=== 网络技术 ==="
Get-ChildItem "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post" -Recurse -Filter "index.md" |
    Select-String -Pattern '"网络技术"' | Measure-Object | Select-Object -ExpandProperty Count

Write-Host "=== 阅读随记 ==="
Get-ChildItem "c:\Users\Ray\Documents\GitHub\Ranch007.github.io\content\post" -Recurse -Filter "index.md" |
    Select-String -Pattern '"阅读随记"' | Measure-Object | Select-Object -ExpandProperty Count
```

Expected: 网络技术 = 12, 阅读随记 = 1。

- [ ] **Step 7: 提交**

```bash
git add -A
git commit -m "feat: 迁移文章分类到新分类体系

- 日常折腾/网安笔记/网工笔记 → 网络技术 (12 篇)
- 痴人呓语 → 阅读随记 (1 篇)

Co-Authored-By: Claude <noreply@anthropic.com>"
```

---

### Task 6: 最终验证

**Files:**
- 不涉及文件修改

- [ ] **Step 1: 检查 git status 干净**

```powershell
git status
```

Expected: `nothing to commit, working tree clean`。

- [ ] **Step 2: 查看 git log 确认所有提交**

```powershell
git log --oneline -5
```

- [ ] **Step 3: Hugo 构建验证（如果有 Hugo CLI）**

```powershell
hugo --quiet 2>&1
```

Expected: 无错误退出。

---

### 文章路径速查

以下为所有 13 篇需迁移文章的完整路径，供验证时对照：

**日常折腾 → 网络技术:**
1. `content/post/2026-05-28/PowerShell-PROFILE-实用手册/index.md`
2. `content/post/2024-11-09/【转载】Windows 下包管理器 Scoop 的安装与使用/index.md`
3. `content/post/2024-11-09/VS Code 配置 C & C++ 编程运行环境/index.md`
4. `content/post/2024-10-20/【博客装修日记】Hugo + Stack + Giscus魔改美化 细节满满/index.md`
5. `content/post/2024-10-18/【Free】使用Hugo搭建GitHub私人博客/index.md`

**网安笔记 → 网络技术:**
6. `content/post/2024-11-19/APP抓包之 Burpsuite+MuMu模拟器12抓包/index.md`
7. `content/post/2024-11-08/掌握 Docker魔法：Windows 11 平台上的完美容器部署终极指南/index.md`
8. `content/post/2024-11-02/超详细的wsl2教程：windows上的linux子系统/index.md`
9. `content/post/2024-10-22/Win11使用JEnv管理多版本jdk/index.md`

**网工笔记 → 网络技术:**
10. `content/post/2024-11-12/GNS3懒人版-2.2.45安装部署详细教程/index.md`
11. `content/post/2024-11-11/【HCIA 路由与交换 Wakin 谢Sir】TCP-IP网络模型/index.md`
12. `content/post/2024-11-11/【HCIA 路由与交换 Wakin 谢Sir】数通网络基础/index.md`

**痴人呓语 → 阅读随记:**
13. `content/post/2024-11-11/有种一语惊醒梦中人的宿命感/index.md`
