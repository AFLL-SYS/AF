# 工控实验手册：OpenPLC + Modbus + Python（第 3 周用）

> 目标：零硬件完成"PLC → Modbus → Python 采集/改值"，这是你 OT 方向的第一个硬实验。
> 全部在本机进行，不碰任何真实设备。

## 1. 装 OpenPLC（10 分钟）

1. openplcproject.com → Download → 下载 **OpenPLC Runtime for Windows** 安装包
2. 安装后启动，浏览器打开 `http://localhost:8080`
3. 默认账号密码：`openplc` / `openplc`

## 2. 写第一个 PLC 程序（20 分钟）

1. 下载安装 **OpenPLC Editor**（同官网），新建项目
2. 程序类型选 ST（结构化文本），main 里写：

```st
PROGRAM main
VAR
    sensor AT %IX0.0 : BOOL;      (* 传感器输入，接到 %IX0.0 *)
    motor  AT %QX0.0 : BOOL;      (* 电机输出，接到 %QX0.0 *)
END_VAR
motor := sensor;                  (* 最简单逻辑：传感器动，电机就动 *)
END_PROGRAM
```

3. 生成程序 → 回 Runtime 网页 → 上传
4. 看到 Running 状态 = 你的第一个"PLC"在跑了

## 3. 用 Python 读写（pymodbus）

装库：`pip install pymodbus`（用你 78 目录的 .venv）

```python
from pymodbus.client import ModbusTcpClient

c = ModbusTcpClient("127.0.0.1", port=502)
c.connect()

# 实验一：读线圈（%QX0.0 对应 coil 地址 0）
rr = c.read_coils(0, 8)
print("线圈状态:", rr.bits)

# 实验二：写线圈——这就是"远程控制 PLC 输出"（攻击演示，只对自己的 OpenPLC）
c.write_coil(0, True)
print("已把第一个线圈置位")

# 实验三：读写保持寄存器（%QW0 对应地址 0）
c.write_register(0, 123)
print("寄存器0 =", c.read_holding_registers(0, 1).registers)

c.close()
```

**地址映射**（OpenPLC 官方文档为准）：`%QX0.0`→线圈 0，`%IX0.0`→离散输入 0，`%QW0`→保持寄存器 0，`%IW0`→输入寄存器 0。

## 4. 三个实验做完，写进打卡

1. **读**：Python 读 8 个线圈，能对上程序里的状态
2. **改**：Python 把线圈/寄存器改成危险值 → **体会一下：没有认证、没有加密，知道地址就能写——这就是工控安全的起点**
3. **采**：写一个循环，每 1 秒读一次寄存器，写入 CSV（加上 try/except，杀进程再启动也不崩）

## 5. Wireshark 抓 Modbus 包（加餐）

- 过滤器输入 `tcp.port == 502`
- 一边抓一边跑 Python 读写
- 看包里的 Function Code：读线圈=1、写线圈=5、读保持寄存器=3、写=6/16
- **能逐字段讲清一个 Modbus 请求包，就是你这周最大的收获**

## 6. 常见坑

- 连不上 502 端口：确认 OpenPLC Runtime 在运行；换端口或检查防火墙
- 网页打不开：只开 `http://`，不是 https
- pymodbus 报错没有 `read_coils`：pymodbus 3.x 用 `from pymodbus.client import ModbusTcpClient`（新写法），老教程的 `ModbusTcpClient` 直接 import 会失败
- 程序上传报错：Editor 和 Runtime 版本要对得上；先用官网给的示例程序测试

## 7. 周报产出（这就是简历项目）

写 10 行：系统构成（OpenPLC + Modbus TCP + Python 客户端）→ 我做了什么（读/写/采集）→ 发现了什么安全问题（无认证、明文、可任意写）→ 建议（加访问控制、网络隔离）。这份东西以后就是简历上"工控仿真环境安全测试"的雏形。
