# Python 工具脚本模板库（复制即用）

> 所有脚本只用于自己的靶场/仿真。改 URL 就能用的最小模板，先跑通再改。

## 1. 通用骨架（所有脚本开头都这么写）

```python
import sys

def main():
    try:
        # 你的逻辑
        print("开始执行...")
    except KeyboardInterrupt:
        print("\n用户中断")
    except Exception as e:
        print(f"出错：{e}")
        sys.exit(1)

if __name__ == "__main__":
    main()
```

## 2. HTTP 请求（带重试）

```python
import requests

def get(url, retries=3, timeout=10):
    headers = {"User-Agent": "Mozilla/5.0"}
    for i in range(retries):
        try:
            r = requests.get(url, headers=headers, timeout=timeout)
            r.raise_for_status()
            return r
        except requests.exceptions.Timeout:
            print(f"超时，第 {i+1} 次重试...")
        except requests.exceptions.RequestException as e:
            print(f"请求失败：{e}")
            break
    return None

r = get("https://quotes.toscrape.com/")
if r:
    print(r.status_code, len(r.text))
```

POST 版：`requests.post(url, data={"k":"v"})` 表单 / `json={"k":"v"}` JSON / `files={"f": open("a.txt","rb")}` 文件。

## 3. 编码转换（渗透日常）

```python
import base64, urllib.parse, binascii

s = "hello 世界"
print(base64.b64encode(s.encode()).decode())          # 字符串→Base64
print(base64.b64decode(b"aGVsbG8=").decode())         # Base64→字符串
print(urllib.parse.quote(s))                          # URL 编码
print(urllib.parse.unquote("%68%65%6c%6c%6f"))        # URL 解码
print(binascii.hexlify(s.encode()).decode())          # 转 hex
print(bytes.fromhex("68656c6c6f").decode())           # hex 还原
```

## 4. CSV / JSON 读写

```python
import csv, json

# 写 CSV（Excel 打开不乱码的关键：utf-8-sig + newline=""）
rows = [["作者", "名言"], ["A", "xxx"]]
with open("out.csv", "w", newline="", encoding="utf-8-sig") as f:
    csv.writer(f).writerows(rows)

# 读 CSV
with open("out.csv", encoding="utf-8-sig") as f:
    for row in csv.reader(f):
        print(row)

# JSON
with open("data.json", "w", encoding="utf-8") as f:
    json.dump(rows, f, ensure_ascii=False, indent=4)
with open("data.json", encoding="utf-8") as f:
    data = json.load(f)
```

## 5. SQLite 存取

```python
import sqlite3

con = sqlite3.connect("test.db")
con.execute("CREATE TABLE IF NOT EXISTS t (id INTEGER, name TEXT)")
con.execute("INSERT INTO t VALUES (?, ?)", (1, "小明"))   # ? 占位 = 防注入
for row in con.execute("SELECT * FROM t"):
    print(row)
con.commit(); con.close()
```

## 6. 爆破模板（只打自己的靶场）

```python
import requests

url = "http://127.0.0.1/dvwa/login.php"
for pw in ["admin", "password", "123456", "test", "root"]:
    r = requests.post(url, data={"username": "admin", "password": pw,
                                 "Login": "Login"})
    if "Login failed" not in r.text:      # 换成靶场实际的失败特征
        print("爆破成功，密码：", pw)
        break
```

## 7. Modbus 读写（工控采集模板）

```python
from pymodbus.client import ModbusTcpClient
import time, csv

c = ModbusTcpClient("127.0.0.1", port=502)
c.connect()

with open("plc_data.csv", "w", newline="", encoding="utf-8-sig") as f:
    w = csv.writer(f)
    w.writerow(["时间", "寄存器0"])
    for _ in range(10):                     # 采集 10 秒
        rr = c.read_holding_registers(0, 1)
        w.writerow([time.strftime("%H:%M:%S"), rr.registers[0]])
        time.sleep(1)

c.close()
```

## 8. 正则速用

```python
import re
m = re.search(r"flag\{.*?\}", text)      # 抓 flag 的典型写法
print(m.group(0) if m else "没找到")
```

## 9. 多线程下载（进阶，知道即可）

```python
from concurrent.futures import ThreadPoolExecutor
def fetch(url):
    return requests.get(url, timeout=10).status_code
with ThreadPoolExecutor(max_workers=10) as ex:
    codes = ex.map(fetch, urls)
```

## 10. 模板库用法

建一个 `tools.py`，把常用的放进去；每写一个新脚本 `from tools import get` 直接用。三个月后，这就是你自己的武器库，面试时说"我有个自用脚本库"比说"会 Python"强得多。
