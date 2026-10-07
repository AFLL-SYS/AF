# Nmap 与端口扫描手册

> Kali 自带 nmap。**只扫自己的靶机或拿到书面授权的目标**，扫互联网上的真实主机是违法取证的第一步。

## 1. 常用命令速查表

| 命令 | 干什么 | 什么时候用 |
|---|---|---|
| `nmap -sn 192.168.1.0/24` | 只发现存活主机（不扫端口） | 进内网第一步 |
| `nmap -sV 目标IP` | 端口 + 服务版本 | 摸清目标开什么服务 |
| `nmap -sC 目标IP` | 跑默认脚本 | 配合 -sV 用 |
| `nmap -p- 目标IP` | 全端口 1–65535 | 找非常规端口（耗时，谨慎） |
| `nmap -p 80,443,502,3306 目标IP` | 指定端口 | 快速验证 |
| `nmap -sS 目标IP` | SYN 半开扫描（要 root） | 默认最常用 |
| `nmap -sU --top-ports 20 目标IP` | UDP 扫描 | 找 DNS/SNMP 等 UDP 服务 |
| `nmap -O 目标IP` | 猜操作系统 | 信息收集 |
| `nmap -T4 目标IP` | 提速（T0–T5） | 内网靶场用 T4 |
| `nmap -Pn 目标IP` | 跳过 ping 直接扫 | 目标禁 ping 时 |
| `nmap -oA 结果名 目标IP` | 结果存盘（-oN 文本 / -oX XML） | 写报告用 |

## 2. 标准流程（背下来）

**存活 → 端口 → 服务版本 → 脚本深挖**

```bash
nmap -sn 网段                      # 1. 谁活着
nmap -sV -p- --min-rate 2000 目标   # 2. 开什么、什么版本（内网靶场可用）
nmap -sV -sC -p 目标端口 目标        # 3. 脚本深挖
```

## 3. 端口状态怎么看

- **open**：有服务在听（继续深挖）
- **filtered**：被防火墙拦（换端口/换协议/换手法）
- **closed**：没服务（跳过）

## 4. 工控脚本（你的差异化）

```bash
nmap --script modbus-discover -p 502 目标      # 发现 Modbus 设备
nmap --script s7-info -p 102 目标              # 西门子 S7 信息
nmap --script enip-info -p 44818 目标          # EtherNet/IP
nmap --script bacnet-info -p 47808 目标        # 楼宇 BACnet
nmap --script dnp3-info -p 20000 目标          # 电力 DNP3
```

只对你的 OpenPLC（502）和 Conpot（502/102）用——这周就能跑通，截图存进项目。

## 5. 面试高频题

1. **SYN 扫描原理？** 只发 SYN，收到 SYN+ACK 就判定开放、立刻发 RST 断开——不完成三次握手，所以叫"半开扫描"，隐蔽。需要 root 权限（构造原始包）。
2. **为什么扫描要限速？** 目标 IDS 会把高速扫描当攻击；`-T4` 是速度与隐蔽的折中。
3. **filtered 和 open 的区别？** filtered = 防火墙把探测包吃了，不代表没服务。

## 6. 配套练习（第 2 周做）

1. `nmap -sV 你自己的 Kali IP` —— 看自己开了什么（先认清自己）
2. 起 OpenPLC 后 `nmap -p 502 127.0.0.1` + modbus-discover —— 截图
3. 扫 Metasploitable2 靶机，对比 Kali 和靶机的开放端口差异
