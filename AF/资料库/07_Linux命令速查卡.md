# Linux 命令速查卡（每天背 5 条，两周过一遍）

## 文件与目录

| 命令 | 干什么 |
|---|---|
| `ls -la` | 列文件（含隐藏） |
| `cd /var/log` | 进目录 |
| `pwd` | 我在哪 |
| `cp -r 源 目标` | 复制（目录要 -r） |
| `mv 源 目标` | 移动/改名 |
| `mkdir -p a/b/c` | 建多层目录 |
| `touch 文件` | 新建空文件 |
| `cat 文件` | 看文件全文 |
| `less 文件` | 翻页看大文件（q 退出） |
| `head -20 文件` / `tail -20 文件` | 看头/尾 20 行 |
| `tail -f 文件` | 实时跟着看（盯日志用） |
| `grep -rn "关键词" 目录` | 在目录里搜关键词（最常用） |
| `find / -name "*.php" 2>/dev/null` | 全盘找文件 |
| `file 文件` | 看文件真实类型 |
| `du -sh 目录` / `df -h` | 目录多大 / 磁盘剩多少 |

## 权限与用户

| 命令 | 干什么 |
|---|---|
| `chmod 755 文件` | 改权限（r=4 w=2 x=1，755=rwxr-xr-x） |
| `chown 用户:组 文件` | 改属主 |
| `whoami` / `id` | 我是谁 / 我的权限详情 |
| `sudo 命令` | 以管理员跑一条 |
| `sudo -i` | 变成 root |
| `who` / `w` / `last` | 谁在登录 / 登录历史 |
| `useradd 名` / `passwd 名` | 建用户 / 改密码 |

## 进程与服务

| 命令 | 干什么 |
|---|---|
| `ps aux | grep nginx` | 找进程 |
| `top` / `htop` | 看系统负载（q 退出） |
| `kill -9 PID` | 强杀进程（谨慎） |
| `systemctl status 服务` | 看服务状态 |
| `systemctl start/stop/restart 服务` | 启停服务 |
| `systemctl enable 服务` | 设开机自启 |

## 网络

| 命令 | 干什么 |
|---|---|
| `ip a` | 看网卡和 IP |
| `ping 目标` | 通不通 |
| `ss -lntp` | 谁在监听端口（l=监听 n=数字 t=tcp p=进程） |
| `netstat -tlnp` | 同上（老版本） |
| `curl -I 网址` | 只看响应头 |
| `curl -O 地址` | 下载文件 |
| `wget 地址` | 下载文件 |
| `ssh 用户@主机 -p 端口` | 远程登录 |
| `nc -lvnp 端口` | 开个监听（收 shell 用） |

## 日志

| 命令 | 干什么 |
|---|---|
| `tail -f /var/log/auth.log` | 实时看登录日志 |
| `journalctl -u ssh -f` | 看指定服务日志 |
| `journalctl -xe` | 看最近的系统日志 |

## 压缩打包

| 命令 | 干什么 |
|---|---|
| `tar -czvf 包.tar.gz 目录` | 打包压缩 |
| `tar -xzvf 包.tar.gz` | 解包 |
| `zip -r 包.zip 目录` / `unzip 包.zip` | zip 打包/解压 |

## 文本处理

| 命令 | 干什么 |
|---|---|
| `grep 关键词 文件` | 搜行 |
| `wc -l 文件` | 数行数 |
| `sort 文件 \| uniq -c` | 排序 + 去重统计 |
| `diff 文件1 文件2` | 对比两个文件 |
| `sed -i 's/旧/新/g' 文件` | 批量替换 |

## 软件包（Debian/Kali 系）

| 命令 | 干什么 |
|---|---|
| `apt update` | 更新软件源 |
| `apt upgrade` | 升级已装软件 |
| `apt install 包名` | 装软件 |
| `apt remove 包名` | 卸载 |
| `dpkg -l | grep 关键词` | 查装了什么 |

## 终端技巧

| 操作 | 作用 |
|---|---|
| `Tab` | 自动补全 |
| `Ctrl+C` | 中断当前命令 |
| `Ctrl+Z` + `fg` | 挂到后台再拉回来 |
| `history` / `!!` | 历史命令 / 重复上一条 |
| `\|` | 管道：把前一个输出给后一个 |
| `>` / `>>` | 输出重定向：覆盖 / 追加 |
| `man 命令` | 查手册（q 退出） |

## 危险命令黑名单（记住长啥样，别执行）

- `rm -rf /` —— 删光系统
- `chmod -R 777 /` —— 全盘权限放开
- `mkfs` —— 格式化磁盘
- `dd if=/dev/zero of=/dev/sda` —— 磁盘清零
- `:(){ :|:& };:` —— fork 炸弹（把机器卡死）

## 用法

每天 5 条：看一眼 → 在 Kali 里亲手敲一遍 → 遮住右边说自己讲。两周过一遍，一个月滚三遍，Linux 就不慌了。
