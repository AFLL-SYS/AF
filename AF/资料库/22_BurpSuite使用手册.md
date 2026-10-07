# Burp Suite 使用手册（面试必问的工具）

## 0. 是什么

Burp Suite 是一个**中间人代理**：你把它架在浏览器和目标网站之间，所有 HTTP(S) 请求都能被它看到、拦下、修改、重发。渗透测试 90% 的时间在它上面。

Kali 自带（菜单里搜 burpsuite），版本 Community 免费够用。

## 1. 配置三步（一次配好）

1. 浏览器代理设成 `127.0.0.1:8080`（Firefox：设置→网络设置→手动代理）
2. 浏览器访问 `http://burp` → 右上角下载 CA 证书
3. 导入浏览器并设为"信任"（Firefox：设置→隐私与安全→证书→查看证书→导入）

**不装证书，所有 HTTPS 页面全是乱码抓不到**——90% 的初学者卡在这。

## 2. Proxy（拦截）

- Intercept is **on**：请求会停在 Burp 里等你
- **Forward**：放行这个包
- **Drop**：丢了这个包
- 改包：直接在界面里改（改参数、改 UA、改 cookie），再 Forward
- HTTP history：所有流量的历史记录，点一条右键可以送到别的模块

## 3. Repeater（手动测试主战场）

单个请求反复改、反复发：

1. 在 HTTP history 里对目标请求右键 → **Send to Repeater**
2. 左边改参数（比如把 `id=1` 改成 `id=1'`）
3. 点 Send → 右边看响应变化

DVWA 的 SQL 注入、命令注入全在这里手动测——**这是面试官认为"会用 Burp"的最低标准**。

## 4. Intruder（批量爆破）

1. 把登录请求 Send to Intruder
2. Positions 标签：把要变的位置用 `§§` 标记（比如密码字段）
3. Payloads 标签：加载字典（Kali 自带 `/usr/share/wordlists/rockyou.txt`）
4. Start attack → 看结果：**响应长度和其他不一样的那条**，多半就是成功

用途：爆破弱口令、枚举目录、批量测参数。

## 5. Decoder / Comparer（辅助）

- Decoder：Base64、URL 编解码（右键 → Send to Decoder）
- Comparer：对比两个响应找差异（比如正确密码 vs 错误密码的响应）

## 6. 实战三连（配 DVWA，做完就算会用）

1. **抓包**：浏览器打开 DVWA 登录页，Burp 里把请求拦下来 → 看懂请求头里的 cookie、参数
2. **Repeater 测 SQLi**：DVWA SQL Injection 的请求送进 Repeater，把 id 参数改成 `1'` 观察报错
3. **Intruder 爆破**：DVWA 登录页用 Intruder，字典放 `admin/password/123456/test`，看哪条响应长度不同

## 7. 常见坑

- 代理没关/拦截开着忘了 → 浏览器一直转圈：把 Intercept 关掉（或 Forward 掉积压的包）
- 抓不到 HTTPS → 证书没导入信任
- Intruder 没反应 → 看请求里是不是缺 cookie（DVWA 要先登录，session 在 cookie 里）
- 中文乱码 → 用响应窗口的编码设置

## 8. 面试怎么答"你会用 Burp 吗"

标准答法：**"会。Proxy 抓包改包、Repeater 手工测参数、Intruder 做爆破，配合 DVWA 做过 SQL 注入和登录爆破的完整验证。"**——说出模块名 + 干过什么事，比说"熟练"强十倍。
