---
title: 重新认识你的 Linux 服务器
published: 2026-07-25
updated: 2026-07-25
description: '你的第一台服务器'
image: ''
tags: ['OS', 'Linux']
category: 'OS'
draft: false
---

> 为什么 PC 不适合作为服务器？
>
> - 外界无法通过 IP 地址访问到你的 PC
>   - 因为多个家用设备共享路由器对外统一使用的 IP 地址，除非使用内网穿透
> - 家用网络的 IP 地址往往是动态的
> - 性能要求会减短 PC 使用寿命
>
> 可以在 [Racknerd](https://www.racknerd.com/)、[BWH](https://bwh81.net)、[Lisa](https://lisahost.com/)、[腾讯云](https://cloud.tencent.com/) 等服务商上选购 VPS / 云服务器，之后可以在 [Namesilo](https://www.namesilo.com/) 上购买域名，并将域名托管到 [Cloudfare](https://www.cloudflare.com/)。
>
> 我参考了 [这篇博客](https://rckin.com/archives/docker-cli_proxy_api)，[VPS 推荐](https://rckin.com/archives/vps-advice-racknerd)，[1C/1G/20G(21.99$/yr)](https://my.racknerd.com/aff.php?aff=14411&pid=952)。如果使用国内服务器，域名必须备案，否则有可能会被封禁。

## 启动

使用硬盘启动时

1. UEFI 固件会在硬盘的 ESP 分区里寻找 EFI 启动文件 (如 `grubx64.efi`) 并启动
2. 引导程序 `grub` 把内核映像 `vmlinuz` 和临时根文件系统映像 `initramfs` 加载进内存
3. `vmlinuz` 自解压成内核 `vmlinux` 到指定地址
4. 内核在内存中建立 `ramfs`/`tmpfs` 文件系统，然后将 `initramfs` 解压并挂载该文件系统至临时根目录 `/`
5. 执行 `/init` 脚本，启动 `udevd` 扫描 PCI/USB 等总线，发现硬件设备 (如 NVMe 控制器) 后将 `initramfs` 里相应的驱动模块 (如 `nvme.ko`, `ext4.ko`) 加载进内核
6. 内核根据启动参数 (`root=UUID...`) 找到系统真正所在分区 (如 `/dev/nvme0n1p2`) 并挂载在 `/sysroot`
7. 执行 `switch_root` 将当前挂载点切换至 `/sysroot`，释放 `initramfs` 占用的内存，执行 `/sbin/init` (即 `systemd`)

## SSH

检查本地是否已安装 SSH

```bash
ssh -V
```

使用 SSH 远程登录服务器

```bash
ssh ubuntu@xxx.xx.xx.xx
```

以往 Telnet 或 FTP 使用明文传输。SSH (Secure SHell，安全外壳协议) 是互联网上最主流、最安全的远程登录和数据传输加密协议。

1. 服务器开机后，`systemd` 直系子进程 `sshd` 调用 `socket()` 和 `accept()` 时刻监听 22 号端口 (处于阻塞状态)。
2. 客户 `A` 在终端输入 `ssh user@IP [-p Port]`，尝试让客户端 `ssh` 与服务器的 `sshd` 建立 TCP 连接。
3. `sshd` 被内核唤醒，执行 `fork()` 后立刻回到 `accept()` 阻塞状态，等待下一位客户。
4. `fork()` 出来的子进程 (记为 `B`) 接管 `socket`，通过 TCP (明文信道) 与客户端交换 SSH 版本信息、支持的加密算法 (KEX)，生成本次会话 ID 和对称加密密钥 (Diffie-Hellman)，接下来的通信将被加密。
5. 由于 TCP 只负责在网络层上的连接，进程 `B` 仍需判断机器背后的用户 `A` 是否有权访问服务器，便 `fork()` 出子进程 `C` 以认证
   1. 进程 `C` 向用户 `A` 发出认证请求，该请求包含了由所有有权用户的公钥组成的列表 (「名单上有你吗？」)
   2. 用户 `A` 查看自己的钥匙扣 `~/.ssh` 是否存在相应私钥 (事前可使用 `ssh-keygen` 生成公私钥)，有则把自己的算法类型和公钥打包成二进制块并发送，无则直接发送密码包（「有！是公钥 xxx」「没有！密码是 xxx」）
   3. 进程 `C` 收到数据包，
      - 对于公钥认证请求，在 `user` 留下的通讯录 `/home/user/.ssh/authorized_keys` (权限必须是 `600`，即仅 `user` 可读写，防止被恶意添加公钥) 里逐一比对公钥。若匹配成功，使用该公钥加密一串随机数，发回给客户端，对 `A` 发起挑战 (「如果真是你，应该能回答我出的题」)。`A` 收到挑战，用相应私钥解密，将随机数答案和当前会话 ID 打包成数字签名（私钥也被用于签名），发回给 `C` (「交卷签字」)。`C` 收到答案，验签得知的确是 `A` 的解答。若回答正确，则通知 `A` 认证成功。
      - 对于密码认证请求，若密码正确，则通知 `A` 认证成功。
6. 认证完毕后，`B` 第二次 `fork()` 出子进程 `D` 以交接。此时 `D` 负责用户交互，`B` 作为监控进程负责网络 I/O。
   1. `D` 调用 `setsid()` 创建新会话
   2. `D` 调用 `openpty()` 申请伪终端，得到文件描述符 `master_fd`(`ptm`) 和 `slave_fd`(`pts`)。`D` 自己持有 master 端，把 slave 端 (`/dev/pts/N`) 交给 `fork()` 出的子进程 `E`。子进程 `E` 继承了父进程 `D` 的会话，但当前会话尚未关联控制终端
   3. 子进程 `E` 调用 `ioctl(slave_id, TIOCSCTTY, 0)` 绑定 slave 端为控制终端，并通过 `dup2()` 将当前进程的 `stdin`/`stdout`/`stderr` 对准控制终端
   4. 子进程 `E` 查看在 `/etc/passwd` 里 `A` 配置的 shell `A:x:1000:1000::/home/A:/bin/bash`，然后 `execve("/bin/bash")`，shell 此时的控制终端就是 `/dev/pts/N`
7. 循环：
   1. 负责用户 I/O 的父进程 `B` 收到来自 socket 的经过加密的用户 `A` 的键盘事件，解密后交给子进程 `D`
   2. 子进程 `D` 将键盘事件 `write(master_fd)` 直接写入 `master` 端，之后 TTY 子系统产生的回显写回 master 端，交由 `B` 通过网络传回客户端，`A` 的终端显示与 `/dev/pts/N` 一致
   3. 在此期间，子进程 `D` 还会监听窗口大小变化 (SSH 协议允许客户端发送窗口改变请求)，调用 `ioctl(master_fd, TIOCSWINSZ, &winsize)` 更新 slave 端尺寸，内核自动向对应会话的前台进程组发送 `SIGWINCH`
8. 客户端网络断开时，`D` 关闭 `master_fd`，内核 PTY 层产生 hangup，slave 端会话首进程 (shell) 收到 `SIGHUP`，进而清理整个会话，所有远程作业被终止

输入密码后，完成登录

```text
The authenticity of host 'xxx.xx.xx.xx (xxx.xx.xx.xx)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'xxx.xx.xx.xx' (ED25519) to the list of known hosts.
ubuntu@xxx.xx.xx.xx's password:
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-124-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Jul 21 09:41:31 PM CST 2026

  System load:  0.01               Processes:             136
  Usage of /:   13.7% of 39.26GB   Users logged in:       1
  Memory usage: 9%                 IPv4 address for eth0: 10.1.0.11
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

144 updates can be applied immediately.
103 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

7 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


Last login: Tue Jul 21 21:23:16 2026 from xxx.xxx.xxx.xxx
```

```console
$ pwd
/home/ubuntu
$ id
uid=1000(ubuntu) gid=1001(ubuntu) groups=1001(ubuntu),4(adm),20(dialout),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),101(lxd),1000(netdev)
```

若要延长 SSH 连接时间，修改 `/etc/ssh/sshd_config`：

```sshconfig
# 取消注释，保持 TCP 连接活跃，每 10 秒发出信号，5 次没响应就断开连接
TCPKeepAlive yes
ClientAliveInterval 10
ClientAliveCountMax 5
```

让 `ubuntu` 能通过 SSH 连接上 Github 仓库

```console
$ ssh-keygen -t ed25519 -C "user@email.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/nisemono/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/nisemono/.ssh/id_ed25519
Your public key has been saved in /home/nisemono/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx user@email.com
The key's randomart image is:
+--[ED25519 256]--+
|     xxxxxxx     |
|  xxxxxxxxxxx    |
| xxxxxxxxxxx     |
|xxxxxxxxxxxxx    |
|xxxxxxxxxxxx     |
|xx xxx   xx      |
|xxx              |
| x               |
|                 |
+----[SHA256]-----+
$ cat ~/.ssh/id_ed25519.pub
ssh-ed25519 XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX user@email.com
$ ssh -T git@github.com
The authenticity of host 'github.com (20.205.243.166)' can't be established.
ED25519 key fingerprint is SHA256:XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Hi user! You've successfully authenticated, but GitHub does not provide shell access.
```

> `-T` 意思是不需要交互式命令行界面，即不申请 PTY。用户只想传输数据或执行一条命令，然后立刻结束连接。`git@github.com` 的 `github.com` 是 Github 主机的域名，`git` 是该主机的用户。

事先说明：远程仓库已经放好了网页文件

```bash
git init
git add .
git status
git commit -m "feat: add tiny web"
git remote add origin git@github.com:user/tiny-web.git
git branch # 查看当前分支名，注意 master/main
git push origin master:master # 本地分支:远程分支
```

## 目录

系统级别的目录

| 目录 | 说明 |
| - | - |
| `/` | 根目录 |
| `/boot/` | 启动目录 (EFI 分区)，内含 `vmlinuz`，`initramfs`，`grub` 等 |
| `/bin/` | 普通工具，一般软链至 `/usr/bin/` |
| `/sbin/` | 系统工具，一般软链至 `/usr/bin/` |
| `/lib/` | 内核模块，基础 C 动态链接库 (如 `libc.so`)，软链至 `/usr/lib` |
| `/etc/` | 系统全局配置，与用户个人配置 `~/.config/` 相对 |
| `/usr/` | 系统资源，如命令、库、软件 |
| `/var/` | 持久化可变数据，如系统日志 `/var/log/`，网页 `/var/www/` |
| `/tmp/` | 临时文件，系统重启自动清空 |

内核接口以目录形式呈现

| 接口 | 说明 |
| - | - |
| `/proc/` | 进程信息和内核参数 (如 `cpuinfo`，`meminfo`) |
| `/sys/` | 硬件设备信息 (如总线、驱动、电源管理) |
| `/dev/` | 设备文件 (如硬盘 `sda`、终端 `tty`、黑洞 `null`、随机数 `random`) |

用户级别的目录

| 目录 | 说明 |
| - | - |
| `/home/<user>/` | 用户 `user` 的家目录 `~` |
| `/root/` | 管理员 `root` 的家目录 |
| `/media/` | 外设自动挂载点 (U 盘、现代桌面环境) |
| `/mnt/` | 手动挂载点 |

外部目录

| 目录 | 说明 |
| - | - |
| `/opt/` | 第三方闭源软件 (如 Google Chrome，Oracle JDK，WeChat) |
| `/srv/` | 本机服务提供的数据 (如 Web 网页根目录、FTP 文件仓库) |

在 `/etc/` 里常见

| 文件或目录 | 说明 |
| - | - |
| `/etc/fstab` | 文件系统挂载表，定义开机时自动挂载哪些分区 (如硬盘，U 盘) |
| `/etc/systemd` | `systemd` 配置，含默认启动级别、用户限额、计时器配置 |
| `/etc/passwd` | 用户数据表，含用户名，`uid`，`gid`，家目录，默认 shell |
| `/etc/group` | 用户组数据表，含组名，`gid`，组内用户 |
| `/etc/shadow` | 用户加密密码，有效期 |
| `/etc/sudoers` | `sudo` 权限表 |
| `/etc/hostname` | 主机名 |
| `/etc/hosts` | 本地静态 DNS 解析表，优先级高于公网 DNS |
| `/etc/resolv.conf` | 指定首选 DNS 服务器 |
| `/etc/network/` | 网卡配置，含 IP，子网掩码，网关 |
| `/etc/ssh/` | SSH 配置，客户端/服务端配置 `ssh_config`/`sshd_config` |
| `/etc/apt/` | 软件源仓库地址，定义 `apt` 从哪里下载软件包 |
| `/etc/<app>` | 软件配置，与软件名同名，如 `nginx`，`docker`，`mysql` |

在 `/usr/` 里常见

| 目录 | 说明 |
| - | - |
| `/usr/bin/` | 普通工具，如 `ls`，`cat`，`git` |
| `/usr/sbin/` | 系统工具，如分区工具 `fdisk`/`gdisk`，服务任务管理 `systemstl` |
| `/usr/lib/` | 库文件，为 `bin` 和 `sbin` 的程序提供运行支持 |
| `/usr/share/` | 只读共享数据，如字体、主题素材、文档、区域、i18n |
| `/usr/local/` | 不使用包管理手动编译安装路径，含子结构 `./bin/`，`./lib/` |
| `/usr/include/` | C/C++ 标准头文件默认目录 |

在 `/var/` 里常见

| 目录 | 说明 |
| - | - |
| `/var/log/` | 系统所有服务的日志文件，如 `syslog`，`nginx/`，`mysql/` |
| `/var/lib/` | 应用的持久化数据，如 `dpkg/`，`docker/` |
| `/var/cache/` | 应用缓存，如 `.deb` 软件包，字体缓存 |
| `/var/www/` | 网页文件 |

## apt

安装常见应用

```bash
sudo apt update
sudo apt install fish neofetch btop fzf tree
```

```bash
sudo apt update # 刷新软件仓库源列表
sudo apt upgrade # 升级所有已安装软件
apt list --installed # 查看已安装软件
apt search <关键词> # 在软件仓库中搜索
apt show <软件名> # 展示软件描述、版本、依赖关系、大小
sudo apt install <软件名> # 安装软件
sudo apt remove <软件名> # 卸载软件 (保留配置文件)
sudo apt purge <软件名> # 卸载软件 (不保留配置文件)
sudo apt autoremove # 卸载无用软件
sudo apt install --reinstall <软件名> # 重装软件
```

## 用户

```bash
sudo useradd -m -s /usr/bin/fish nisemono
sudo passwd nisemono
sudo usermod -aG sudo nisemono
su -l nisemono
```

```console
$ id
uid=1002(nisemono) gid=1003(nisemono) groups=1003(nisemono),27(sudo)
```

1. `useradd`：`-m`/`-M` 创建/不创建家目录，`-s` shell，`-g` 主组 (默认同名)，`-G` 附加组 (默认只有主组)
2. `usermod`：修改用户属性，参数与 `useradd` 一致，`-Ga` 追加附加组，`-g` 主组的修改需要重新登录才能生效
3. `passwd`：修改密码，执行者只能是 `root` 或 `sudo` 提权后的用户
4. `su`：切换用户 (默认切换至 `root`)，需要输入目标用户密码，不加 `-l` 会停留在当前路径并且环境变量不变，执行 `exit` 回到原用户

> 默认管理组的命名不统一：Debian 系 (Debian、Ubuntu、Linux Mint) 是 `sudo`，而 Red Hat 系 (RHEL、CentOS、Fedora) 和传统的 Unix (如 BSD) 是 `wheel`
>
> 可以通过查看 `/etc/sudoers` 确认
>
> ```sudoers
> %sudo ALL=(ALL:ALL) ALL
> %wheel ALL=(ALL:ALL) ALL
> ```

虽然你不应该知道服务器的 `root` 密码，但服务提供商为你准备了 `sudo` 组里的一个用户及其密码。`sudo` 是被系统认可的用户组，使用 `sudo` 命令就可以按照 `/etc/sudoers` 里 `sudo` 组规定的权限进行操作。

- `-u <username>` 以指定用户的身份，默认为 `root`
- `-i` 登录指定用户 (同 `su -l`，但输入的是自己的密码)
- `-s` 登录指定用户 (同 `su`，但输入的是自己的密码)
- `-E` 保留当前用户的环境变量
- `-l` 查看当前用户权限

比如 `nisemono` 以 `ubuntu` 的身份将 Github 仓库 `tiny-web` 克隆至 `/home/ubuntu/tiny-web/`

```bash
> sudo -u ubuntu git clone git@github.com:user/tiny-web.git /home/ubuntu/tiny-web
Cloning into '/home/ubuntu/tiny-web'...
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 5 (delta 0), reused 5 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (5/5), done.
> sudo -u ubuntu tree -L 2 /home/ubuntu/
/home/ubuntu/
└── tiny-web
    ├── index.html
    ├── script.js
    └── style.css

2 directories, 3 files
```

`/etc/sudoers` 的配置示例：

```sudoers
Host_Alias office-pc = pc1, pc2, pc3, 192.168.1.11

alice ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
bob ALL=(ALL) ALL, !/usr/bin/passwd, !/usr/sbin/visudo
charlie ALL=(ALL) SETENV: /usr/bin/python3
%dba db=(postgres) /usr/bin/psql
%dev office-pc=(:www-data) NOEXEC: /usr/bin/vim /var/www/html/*

# 仅允许用户 alice 在任何主机=(以任何身份) 免密重启 nginx
# 允许用户 bob 在任何主机=(以任何身份) 做任何事，除了修改密码和 sudo 配置
# 仅允许用户 charlie 在任何主机=(以任何身份) 运行 python3 (允许带上自己的环境变量)
# 仅允许 dba 组成员 在主机 db=(以用户 postgres 的身份) 执行 sql
# 仅允许 dev 组成员 在主机 pc1,pc2,pc3,192.168.1.11=(以 www-data 组的权限) 使用 vim (不允许使用内部的 execve 系统调用，防止 getshell 逃逸) 编辑 /var/www/html/ 里的所有文件，此时用户身份不变，但组权限生效
```

```sudoers
# 别名
User_Alias      WEBADMINS = alice, bob, %devops
Cmnd_Alias      WEB_CMDS = /usr/bin/systemctl restart nginx, /usr/bin/systemctl reload nginx
Runas_Alias     WEBUSER = www-data

# 授权部分
WEBADMINS ALL=(WEBUSER) WEB_CMDS
```

## 身份与权限

Linux 有三种身份，每种身份拥有独立的权限设置：

- `u` (所有者，User)：文件的创建者或通过 `chown` 指定的用户
- `g` (所属组，Group)：文件所属的用户组，组内所有成员共享这组权限
- `o` (其他人，Others)：既不是所有者，也不属于文件所属组的其他所有用户

Linux 有三种权限：

| 权限 | 文件 | 目录 |
| - | - | - |
| `r`(读) | `cat`，`less`，`more`，`head`，`tail`，`grep`，`awk`，`sed`(只读)，`vim`(只读)，`<`(重定向输出)，`cp` | `ls` 浏览 (不允许长格式 `-l`) |
| `w`(写) | `>`(覆盖)，`>>`(追加) | 创建/删除/重命名目录下的文件 |
| `x`(执行，进入) | 将二进制文件加载进内存并运行 | `cd`，解析文件路径，`stat` (inode 元数据：查看大小，修改时间，权限设置)， |

在 `ls -l` 的输出里，权限的文本格式形如 `-rwxr-xr--`，含 10 个字符：

- `-`/`d` (第 1 位)：文件/目录
- `rwx` (第 2-4 位)：对 `u` (所有者) 的权限
- `r-x` (第 5-7 位)：对 `g` (所属组) 的权限
- `r--` (第 8-10 位)：对 `o` (其他人) 的权限

三种身份的权限也可以编码成 3 位八进制数，高位对应 `u`，中位对应 `g`，低位对应 `o`

- `r`：`4`，即 `0b100`
- `w`：`2`，即 `0b010`
- `x`：`1`，即 `0b001`

因此 `rwxr-xr--` 也可以编码成 `754` (八进制)

| 常见场景 | 权限 |
| - | - |
| 常规文件 | `644` (`-rw-r--r--`)，不可执行，组和其他人只读 |
| 常规目录 | `755` (`drwxr-xr-x`)，组和其他人能 `cd` 并 `ls`，但不能 `touch` 或 `rm` |
| 常规二进制 | `755` (`-rwxr-xr-x`)，可执行，仅所有者可写 |
| 系统二进制 | `555` (`-r-xr-xr-x`)，可执行，所有身份不可写 |
| 敏感文件 | `600`，SSH 私钥，`.env` 环境变量 (含 API-key)，数据库配置文件 |

修改文件/目录权限：

```bash
# 符号模式，+(添加)，-(移除)，=(只权限)
# 所有者可执行，所属组不可写，其他人只读
chmod u+x,g-w,o=r file.txt
# 所有身份可读
chmod a+r file.txt

# 数字模式，三位八进制赋值
# 所有者=读写执行，所属组=读执行，其他人无权限
chmod 750 folder
```

比如让 `ubuntu` 用户的家目录为其他人 `o` 添加进入权限 `x`

```bash
sudo chmod o+x /home/ubuntu
```

## 文本处理

### grep

`grep` 是 Linux 中常用的文本搜索工具

```bash
# 忽略大小写，完整单词匹配，显示匹配行号
grep -i -w -n error log.txt > error.txt

apt list\
# 排除匹配成功行
| grep -v i386\
# 启用扩展正则表达式
| grep -v -E "^lib.*\[.*?installed.*?\]$"\
# 只显示匹配成功行数
| grep python | grep -c clang
# 给文本标行号 (也可以 cat -n)
| nl

# 递归搜索
grep -r -E - "flag{.*?}" /home/

# 指定文件后缀，显示前后行
grep -r -w --include="*.py" -C 5 "def" ./src/
```

### less

`less` 是 Linux 中常用的文本查看器，适用于查看不支持滚屏的终端容纳不下的输出内容

```bash
less /var/log/syslog
ps aux | less
```

- `q` 退出，`h` 帮助
- `d`/`u` 翻半屏，`SPACE`/`b` 翻一屏，`j`/`k` 翻一行，`←`/`→` 左右滚动
- `g`/`G` 首尾，`114g` 行号
- `/`/`?` 向下/上搜索，`n/N` 下/上一个，`-i` 大小写敏感，`[ESC]u` 取消高亮
- `v` 切换至 `vi` 编辑，保存退出后回到 `less`

### head/tail

`head` 和 `tail` 用于查看文件首尾内容 (默认 10 行)，避免终端刷屏

`-n` 控制行数，`head -n -9` 舍弃末尾 9 行，`tail -n +7` 从第 7 行开始保留

```bash
head -n 20 sys.log
apt list | tail -n +100 | head -n 50
```

## Nginx

```bash
sudo apt update
sudo apt install nginx
```

```console
$ systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-07-27 09:33:32 CST; 1min 1s ago
       Docs: man:nginx(8)
    Process: 1848505 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 1848507 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 1848539 (nginx)
      Tasks: 5 (limit: 4369)
     Memory: 3.8M (peak: 8.1M)
        CPU: 25ms
     CGroup: /system.slice/nginx.service
             ├─1848539 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─1848541 "nginx: worker process"
             ├─1848542 "nginx: worker process"
             ├─1848543 "nginx: worker process"
             └─1848544 "nginx: worker process"
```

现在服务器已经正在监听 80 端口，在本地浏览器输入 `http://xxx.xx.xx.xx/` (注意不是 `https`) 就可以使用 HTTP 协议访问服务器 `nginx` 的欢迎界面。

> Linux 有 65535 个端口，前 1024 个端口被系统保留，常见端口如下：
>
> | 端口 | 协议/服务 | 解释 |
> | - | - | - |
> | 21 | FTP | 文件传输 (明文) 协议 |
> | 22 | SSH | 你正在使用 22 端口连接着这台服务器 |
> | 53 | DNS | 域名解析服务 |
> | 80 | HTTP | 网页传输 (明文) 协议 |
> | 443 | HTTPS | 网页传输 (SSL/TLS 加密) 协议 |
> | 3306 | MySQL | 最流行的关系型数据库 |
> | 5432 | PostgreSQL | 更稳健的开源关系型数据库 |
> | 6379 | Redis | 内存缓存/键值对数据库 |
> | 27017 | MongoDB | 非关系型文档数据库 (NoSQL) |
>
> 查看服务器此刻在监听哪些端口
>
> ```bash
> sudo ss -tulnp
> ```
>
> `-t`/`-u` TCP/UDP，`-n` 显示数字端口，`-p` 显示进程名

`/etc/nginx/` 常见：

| 文件/目录 | 功能 |
| - | - |
| `nginx.conf` | 配置总入口，Nginx 启动时唯一直接读的文件 |
| `sites-available/` | 所有网站配置 |
| `sites-enabled/` | 当前启用的网站配置，软链至 `sites-available/` |
| `conf.d/` | 额外的全局配置片段，`nginx.conf` 默认会 include |
| `snippets/` | 可复用的配置片段，需要时自己 include 进来 |
| `modules-available/` | 可用的动态模块 |
| `modules-enabled/` | 启用的动态模块 |
| `mime.types` | 文件扩展名到 MIME 类型的映射表，告诉浏览器返回的是 html、png 还是别的 |
| `proxy_params` / `fastcgi_params` / `fastcgi.conf` / `scgi_params` / `uwsgi_params` | 反向代理到后端应用时常用的参数片段，需要时 include 即可 |

> `ls -l` 查看 `sites-enabled/` 的软链情况

`nginx.conf` 配置 Nginx：

```nginx
user www-data; # 以 www-data 组的身份访问本机的网页文件
worker_processes auto; # 工作进程数与 CPU 核数一致
pid /run/nginx.pid; # 进程 ID
error_log /var/log/nginx/error.log; # 日志位置
include /etc/nginx/modules-enabled/*.conf; # 导入动态模块

# 并发处理参数
events {
    # 单进程处理的最高连接数
    worker_connections 768;
}

# 所有跟 HTTP 服务相关的设置
http {
    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;

    # 扩展名在 mime.types 没有对应规则时，按二进制流返回
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    access_log /var/log/nginx/access.log;

    gzip on; # Gzip 自动压缩，省流量

    # 导入配置
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

> 注意：`www-data` 不能改成高权限用户 (如 `ubuntu`，`root` 等)，否则黑客攻破 Nginx 后就能以该用户的权限行事，他可以随意修改你的代码、删除文件，甚至执行 `sudo` 提权控制整台服务器

欢迎界面对应了服务器上的一个 HTML 文件，你可以在 `/etc/nginx/sites-available/default` 里找到它的位置

```nginx
server { ... root /var/www/html; ... }
```

`/etc/nginx/sites-available/default` 的结构：

```nginx
# 一个 server 块 = 一个网站
server {
    # 监听 80 端口
    # 没有匹配其他 server 块的时候，作为默认 server 块
    listen 80 default_server;
    listen [::]:80 default_server; # IPv6

    # 用户访问到该端口时，去指定目录，按顺序优先找指定文件返回
    # 记得给 www-data 组 (nginx 配置文件里设置的) 开目录权限
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    # 对所有域名都响应
    server_name _; # example.com

    # 路由规则
    location / {
        # 先按用户要的路径找文件，找不到就当作目录找，再找不到就返回 404
        try_files $uri $uri/ =404;
    }
}
```

配置修改完毕后，让配置生效，成功上线

```bash
sudo nginx -t # 语法检查，有错的话一定要在 reload 前改对，否则会挂
sudo systemctl reload nginx # 不是 restart，不会断连
```

> 服务器上的代码应当是 GitHub 仓库的镜像：所有改动都应该在本地做，通过 push/pull 同步过去，不要在服务器上手改文件
