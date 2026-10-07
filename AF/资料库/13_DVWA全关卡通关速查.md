# DVWA 全关卡通关速查（12 个模块 × 4 档难度）

> 目标：Security 从 Low 一路打到 Impossible。每过一档，把"过滤了什么"记进打卡。
> 详细步骤看 [渗透实验手册_DVWA四类漏洞.md](12_渗透实验手册_DVWA四类漏洞.md) 和 [Web漏洞进阶实验手册.md](14_Web漏洞进阶实验手册.md)，这里是速查表。

## 1. Brute Force（爆破）

| 难度 | 怎么过 |
|---|---|
| Low | 随便爆：hydra/Intruder，看响应长度 |
| Medium | 加了 sleep(2)——爆破变慢而已，照爆 |
| High | 加了 CSRF token——Intruder 用 Recursive Grep 提取 token，或手测 |

## 2. Command Injection（命令注入）

| 难度 | 怎么过 |
|---|---|
| Low | `127.0.0.1; whoami` |
| Medium | `;` 和 `&&` 被过滤 → 用 `\|` |
| High | 过滤更多 → 看源码，找漏网（如 `\|` 后带空格） |

## 3. CSRF

| 难度 | 怎么过 |
|---|---|
| Low | 构造链接/页面，受害者点开即改密码 |
| Medium | 校验 Referer → 抓包看它怎么查的，改 URL 路径 |
| High | 加了 token → 概念题：token 是正解 |

## 4. File Inclusion（文件包含）

| 难度 | 怎么过 |
|---|---|
| Low | `?page=../../../../etc/passwd` |
| Medium | `../` 被过滤 → 用 `..//` 双写绕过 |
| High | 限定 `file` 开头 → `?page=file:///etc/passwd` 或 `file4.php` 技巧 |

## 5. File Upload（文件上传）

| 难度 | 怎么过 |
|---|---|
| Low | 直接传 `shell.php` |
| Medium | 改 `Content-Type: image/jpeg` |
| High | 图片马（`GIF89a` + php），或绕过扩展名过滤 |

## 6. SQL Injection

| 难度 | 怎么过 |
|---|---|
| Low | `' OR '1'='1` 万能密码 + union 拖库 |
| Medium | 下拉框数字型 → Burp 抓包改参数再注入 |
| High | 手注仍可行，只是查不出数据时换方法（limit 绕过） |

## 7. SQL Injection (Blind) 盲注

| 难度 | 怎么过 |
|---|---|
| Low | 布尔：`1' and 1=1#` vs `1' and 1=2#` |
| Medium/High | 时间盲注：`1' and sleep(5)#` |

## 8–9. XSS Reflected / Stored

| 难度 | 怎么过 |
|---|---|
| Low | `<script>alert(1)</script>` |
| Medium | `script` 被过滤 → `<img src=x onerror=alert(1)>` |
| High | 更严过滤 → 大小写/编码变体，或看源码找漏 |

## 10. Weak Session IDs

观察按钮点击生成的 session：Low 是 +1 递增（可预测别人的会话），Medium 是时间戳，High 是加密串——**目的是让你理解"会话 ID 必须不可预测"**。

## 11. Insecure CAPTCHA

验证码在**客户端**校验：Burp 抓包，直接把 `step=1` 改成 `step=2` 跳过验证——原理：客户端校验不可信。

## 12. CSP Bypass / Open Redirect

- CSP：给页面注入的 JS 被 Content-Security-Policy 拦了，试不满足的绕过或只做概念理解
- Open Redirect：登录后跳转参数没校验，`?redirect=https://evil.com`——原理：跳转目标要白名单

## 通关标准

12 个模块全部到 Impossible 或最高档（个别 Impossible 允许只做"看源码理解为什么打不动"）。
**每关记一行：过滤了什么 + 我怎么绕的。** 这张表做完，Web 安全基础就毕业了。
