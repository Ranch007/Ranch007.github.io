---

title: "PowerShell $PROFILE 实用手册"
slug: "PowerShell-PROFILE-实用手册"
description: "从 PSReadLine 语法高亮、智能补全到自定义提示符、别名快捷函数，再到 Terminal-Icons 图标集成与 Oh My Posh 主题配置，一篇带你打造高效 PowerShell 工作环境的实战指南"
date: "2026-05-28T10:00:00+08:00"
image: maxresdefault.jpg
math: 
license: 
hidden: false
draft: false 
categories: ["网络技术"]
tags: ["PowerShell", "Windows", "环境"]

---

---

## 前言

> [!WARNING]
>
> **本篇文章内容，主要针对 PowerShell 实用性配置，不涉及终端美化，请酌情食用。**

`$PROFILE` 是 PowerShell 中一个非常重要的自动变量，它存储了 PowerShell 配置文件（Profile）的路径。通过编辑它，你可以打造一个完全符合自己习惯的终端环境。

下面是一段快速预览——将以下内容保存到你的 `$PROFILE` 中，立即体验语法高亮、智能补全、自定义提示符和文件图标：

```powershell
# 1. 语法高亮与智能提示补全（默认自带，可在此优化）
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete
Set-PSReadLineOption -PredictionSource HistoryAndPlugin
Set-PSReadLineOption -PredictionViewStyle ListView
Set-PSReadLineOption -Colors @{
    Command = 'Yellow'
    Number = 'Magenta'
}

# 2. 自定义提示符（显示时间与当前路径）
function prompt {
    $date = Get-Date -Format 'HH:mm:ss'
    Write-Host "[$date] " -NoNewline -ForegroundColor Gray
    Write-Host "$(Get-Location) " -NoNewline -ForegroundColor Cyan
    return "> "
}

# 3. 文件Icons
Import-Module -Name Terminal-Icons
```

## 如何配置你的 $Profile

### 查看路径

在终端输入 `echo $PROFILE` 查看当前配置文件的具体路径。

### 创建文件

如果文件不存在，使用以下命令创建（注意如果 Documents 目录被 OneDrive 同步，建议将路径指向非同步目录）：

```powershell
if (!(Test-Path $PROFILE)) { New-Item -Type File -Path $PROFILE -Force }
```

### 编辑配置

在终端输入 `notepad $PROFILE`（或在 VS Code 中输入 `code $PROFILE`）打开并编辑文本。

> [!IMPORTANT]
>
> **PSReadLine**：优化命令历史、自动补全以及智能语法高亮。
>
> **常用路径别名**：通过 `Set-Alias` 将常用长命令或路径设为简写。
>
> **快捷函数**：创建自定义函数（如一键进入工作目录、一键清空终端缓存等）。
>
> **环境变量**：配置全局变量或 API Key，供脚本或工具直接调用。

### 1. 语法高亮与智能提示补全

默认自带，可在 `$PROFILE` 中进一步优化：

```powershell
# ==========================================
# 1. 核心功能：输入字母按 Tab 键，直接弹出命令选单
# ------------------------------------------
# 将 Tab 键功能修改为：输入字母后按 Tab，弹出所有相关命令的网格菜单
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete

# ==========================================
# 2. 核心功能：历史命令智能感知与全列表显示
# ------------------------------------------
# 开启历史与插件预测
Set-PSReadLineOption -PredictionSource HistoryAndPlugin

# 默认将预测显示样式设置为"列表视图(ListView)"，输入时会直接展开下拉列表
Set-PSReadLineOption -PredictionViewStyle ListView

# ==========================================
# 3. 界面美化与颜色自定义（可根据喜好修改）
# ------------------------------------------
# 设置提示列表的颜色（命令为黄色，数字为品红）
Set-PSReadLineOption -Colors @{
    Command = 'Yellow'
    Number = 'Magenta'
}

# （可选）Windows 剪贴板习惯：Ctrl + C 复制，Ctrl + V 粘贴
Set-PSReadLineKeyHandler -Key "Ctrl+v" -Function Paste
Set-PSReadLineKeyHandler -Key "Ctrl+c" -Function Copy
```

### 2. （可选）自定义提示符（显示时间与当前路径）

```powershell
function prompt {
    $date = Get-Date -Format 'HH:mm:ss'
    Write-Host "[$date] " -NoNewline -ForegroundColor Gray
    Write-Host "$(Get-Location) " -NoNewline -ForegroundColor Cyan
    return "> "
}
```

### 3. 常用别名和快捷方式

```powershell
Set-Alias cls Clear-Host
function cdws { Set-Location "C:\你的常用工作目录" }
```

### 4. （可选）显示文件夹 Icons

在 `PowerShell` 中输入以下代码。

第一行代码，从官方信任源下载并安装 `Terminal-Icons` 模块：

```powershell
Install-Module -Name Terminal-Icons -Repository PSGallery -Force -Scope AllUsers
Set-ExecutionPolicy RemoteSigned -Force
```

然后将以下代码粘贴到 `$PROFILE` 文件的**最底部**：

```powershell
# 自动导入终端图标模块
Import-Module -Name Terminal-Icons
```

### 5. （可选）集成 Oh My Posh 主题

需提前安装模块。以下代码仅供参考，详见教程 —— [使用 Oh My Posh 为 PowerShell 或 WSL 设置自定义提示符](https://learn.microsoft.com/zh-cn/windows/terminal/tutorials/custom-prompt-setup)

```powershell
# Oh-My-Posh 配置示例
oh-my-posh init pwsh --config "$HOME\AppData\Local\Programs\oh-my-posh\themes\jandedobbeleer.omp.json" | Invoke-Expression
```

### 6. 配置生效

编辑完成后保存关闭记事本，输入 `. $PROFILE` 立刻应用修改。

### 7. 开启脚本权限（如遇报错）

如果提示无法加载文件，需要以管理员身份运行 PowerShell，并执行策略：

```powershell
Set-ExecutionPolicy RemoteSigned
```

## 拓展

> [!TIP]
>
> 这里博主安装了新的 `Notepad.exe`，但是 PowerShell 中输入 `notepad` 命令依旧使用的是旧的 `notepad.exe`。
>
> 要解决这个问题，需要先"挪走"原有的旧版记事本。但由于它是系统核心文件，直接删除或改名会被系统拒绝。
>
> 按照以下两步操作，先解锁权限并改名，然后再创建链接。

```powershell
#  1. 将旧记事本的所有者修改为当前管理员组
takeown /f "C:\Windows\system32\notepad.exe" /a

#  2. 赋予管理员组对该文件的完全控制权限
icacls "C:\Windows\system32\notepad.exe" /grant administrators:F

#  3. 将旧记事本重命名为备份文件（腾出位置）
Rename-Item -Path "C:\Windows\system32\notepad.exe" -NewName "notepad.exe.bak"
```

```powershell
# 第二个路径可替换为你使用的其他记事本软件路径（如 Notepad3 等）
New-Item -ItemType SymbolicLink -Path "C:\Windows\system32\notepad.exe" -Target "C:\Program Files\WindowsApps\Microsoft.WindowsNotepad_11.2604.5.0_x64__8wekyb3d8bbwe\Notepad\Notepad.exe"
```

## 参考文章

1. [使用 Oh My Posh 为 PowerShell 或 WSL 设置自定义提示符 - Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/terminal/tutorials/custom-prompt-setup)

### 版权信息

本文原载于 [Ranch's Blog](https://ranch007.github.io)，遵循 CC BY-NC-SA 4.0 协议，复制请保留原文出处。

