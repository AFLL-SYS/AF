# Windows 安全速查卡（配 Linux 卡一起背）

> 你以后面对的真实环境一半是 Windows。每天背 3 条，在 Windows 上亲手敲。

## 系统与用户

| 命令 | 干什么 |
|---|---|
| `systeminfo` | 系统信息（版本、补丁——提权先看它） |
| `whoami` / `whoami /all` | 我是谁 / 我的权限与组 |
| `net user` | 列出用户 |
| `net user 用户名 密码 /add` | 建用户（靶场实验用） |
| `net localgroup administrators` | 看管理员组有谁 |
| `ipconfig /all` | 网卡、IP、DNS |

## 进程与网络

| 命令 | 干什么 |
|---|---|
| `tasklist` / `tasklist /svc` | 进程列表 / 带服务 |
| `netstat -ano` | 所有连接 + PID |
| `netstat -ano \| findstr 445` | 看特定端口 |
| `taskkill /F /PID 进程号` | 杀进程 |
| `wmic process get name,parentprocessid` | 进程树（找恶意进程的爹） |

## 启动项与计划任务（后门最爱藏这里）

| 命令 | 干什么 |
|---|---|
| `msconfig` | 图形界面看启动项 |
| `reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run` | 注册表自启 |
| `schtasks /query /fo LIST /v` | 所有计划任务 |
| `schtasks /query /tn "任务名"` | 查单个任务 |

## 日志（应急的核心）

| 命令 | 干什么 |
|---|---|
| `eventvwr.msc` | 事件查看器（图形界面） |
| `wevtutil qe Security /c:50 /f:text` | 最近 50 条安全日志 |
| 4624 / 4625 | 成功登录 / 失败登录（爆破就查 4625 数量） |
| 4688 | 进程创建（看攻击者跑了什么） |

## 防火墙与共享

| 命令 | 干什么 |
|---|---|
| `netsh advfirewall show allprofiles` | 防火墙状态 |
| `netsh advfirewall firewall add rule name="x" dir=in action=allow protocol=TCP localport=80` | 放行端口（靶场用） |
| `net share` | 看开了什么共享 |

## 常用技巧

- 显示隐藏文件/扩展名：资源管理器 → 查看 → 勾选（木马最爱伪装成 `发票.jpg.exe`，不显示扩展名就中招）
- 判断杀软：`wmic /namespace:\\root\securitycenter2 path antivirusproduct get displayname`
- 快速搜文件：`dir /s /b C:\*.php`（在网站目录找 webshell 用）
- PowerShell 版查找：`Get-ChildItem -Recurse -Path C:\inetpub | Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-3)}`（近 3 天被改动的文件）

## 面试常问的 Windows 考点

1. 怎么查开机自启？→ 注册表 Run 键 + msconfig + 计划任务
2. 怎么判断被爆破？→ 安全日志 4625 大量 + 4624 伴随出现
3. 445 是什么？→ SMB 文件共享端口，永恒之蓝就在这（你有 [内网基础手册.md](26_内网基础手册.md)）

## 用法

Linux 卡每天 5 条 + Windows 卡每天 3 条，两周过一遍。两边都会，才叫"系统安全"。
