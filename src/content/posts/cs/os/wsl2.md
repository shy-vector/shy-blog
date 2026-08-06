---
title: WSL2 + Arch Linux
published: 2026-03-15
updated: 2026-03-15
description: '记录 WSL2 + Arch Linux 安装过程，供我以后参阅。'
image: ''
tags: ['OS', 'Linux', 'WSL2']
category: 'OS'
draft: false
---

:::warning
以下内容可能过时，请注意文章最近更新时间．
:::

## 前期准备

> 我正在使用 [Windows 11, version 25H2](https://uupdump.net/selectlang.php?id=db640db1-d7b0-4d44-8bbc-b930ab72865a)，SKU 版本为 Windows 家庭中文版。

首先检查自己的 Windows 是否启用了**虚拟化**：“右击任务栏 —— 任务栏管理 —— 性能 —— CPU”，观察到 “虚拟化：已启用”。

> 大部分电脑默认已开启了虚拟化。如果没有启用，请在 UEFI/BIOS 设置中[手动开启虚拟化](https://support.microsoft.com/en-us/windows/enable-virtualization-on-windows-c5578302-6e43-4b4b-a449-8ced115f58e1)。

在开始菜单中，依次点击 “设置 —— 系统 —— 可选功能 —— 更多 Windows 功能”，进入 “启用或关闭 Windows 功能”，开启 “**适用于 Linux 的 Windows 子系统**” 和 “**虚拟机平台**” 两个功能，确认后需要**重启**你的计算机。

打开 PowerShell，下载 **WSL** (适用于 Linux 的 Windows 子系统，即 **Windows Subsystem for Linux**)：

```ansi
> [35mwsl[0m --update
```

```ansi
请求的操作需要提升。
正在下载: 适用于 Linux 的 Windows 子系统 2.6.3
正在安装: 适用于 Linux 的 Windows 子系统 2.6.3
已安装 适用于 Linux 的 Windows 子系统 2.6.3。
正在检查更新。
已安装最新版本的适用于 Linux 的 Windows 子系统。
```

> ```ansi
> 正在检查更新。
> 已禁止(403)。
> 错误代码: Wsl/UpdatePackage/0x80190193
> ```
>
> 如果在检查更新时出现 “已禁止(403)”，请关闭网络代理功能。

现在我成功安装了[适用于 Linux 的 Windows 子系统 2.6.3](https://github.com/microsoft/WSL/releases/tag/2.6.3)，接着检查 WSL、WSLg、Linux 内核的版本信息：

```ansi
> [35mwsl[0m --version
```

```ansi
WSL 版本: 2.6.3.0
内核版本: 6.6.87.2-1
WSLg 版本: 1.0.71
MSRDC 版本: 1.2.6353
Direct3D 版本: 1.611.1-81528511
DXCore 版本: 10.0.26100.1-240331-1435.ge-release
Windows: 10.0.26200.8037
```

:::tip
你也可以[从 Microsoft Store 下载 WSL](https://learn.microsoft.com/en-us/windows/wsl/install)，这允许您比以前更快地获得 WSL 更新。
:::

查看支持在线安装的 Linux 发行版：

```ansi
> [35mwsl[0m --list --online 
```

```ansi
以下是可安装的有效分发的列表。
使用“wsl.exe --install <Distro>”安装。

NAME                            FRIENDLY NAME
Ubuntu                          Ubuntu
Ubuntu-24.04                    Ubuntu 24.04 LTS
openSUSE-Tumbleweed             openSUSE Tumbleweed
openSUSE-Leap-16.0              openSUSE Leap 16.0
SUSE-Linux-Enterprise-15-SP7    SUSE Linux Enterprise 15 SP7
SUSE-Linux-Enterprise-16.0      SUSE Linux Enterprise 16.0
kali-linux                      Kali Linux Rolling
Debian                          Debian GNU/Linux
AlmaLinux-8                     AlmaLinux OS 8
AlmaLinux-9                     AlmaLinux OS 9
AlmaLinux-Kitten-10             AlmaLinux OS Kitten 10
AlmaLinux-10                    AlmaLinux OS 10
archlinux                       Arch Linux
FedoraLinux-43                  Fedora Linux 43
FedoraLinux-42                  Fedora Linux 42
eLxr                            eLxr 12.12.0.0 GNU/Linux
Ubuntu-20.04                    Ubuntu 20.04 LTS
Ubuntu-22.04                    Ubuntu 22.04 LTS
OracleLinux_7_9                 Oracle Linux 7.9
OracleLinux_8_10                Oracle Linux 8.10
OracleLinux_9_5                 Oracle Linux 9.5
openSUSE-Leap-15.6              openSUSE Leap 15.6
SUSE-Linux-Enterprise-15-SP6    SUSE Linux Enterprise 15 SP6
```

我即将安装 [Arch Linux](https://wiki.archlinux.org.cn/title/Install_Arch_Linux_on_WSL)：

```ansi
> [35mwsl[0m --install archlinux
```

```ansi
正在下载: Arch Linux
正在安装: Arch Linux
已成功安装分发。可以通过 “wsl.exe -d archlinux” 启动它
正在启动 archlinux...
Welcome to the Arch Linux WSL image!

This image is maintained at <https://gitlab.archlinux.org/archlinux/archlinux-wsl>.

Please, report bugs at <https://gitlab.archlinux.org/archlinux/archlinux-wsl/-/issues>.
Note that WSL 1 is not supported.

For more information about this WSL image and its usage (including "tips and tricks" and troubleshooting steps), see the related Arch Wiki page at <https://wiki.archlinux.org/title/Install_Arch_Linux_on_WSL>.

While images are built regularly, it is strongly recommended running "pacman -Syu" right after the first launch due to the rolling release nature of Arch Linux.

Generating pacman keys...
Done

Populating keyring...
Done
[root@shy-pc-windows /]# 
```

此时我便以 `root` 身份进入了 Arch Linux 环境。

:::tip
WSL 可安装并运行多个 Linux 系统。在 PowerShell 中查看已安装的 WSL：

```ansi
> [35mwsl[0m --list
```

```ansi
适用于 Linux 的 Windows 子系统分发:
archlinux (默认值)
```

若想再次启动 WSL，可以在 PowerShell 中打开 “新建标签页” 的下拉菜单，选择想启动的 Linux 发行版后点击启动，进入 Linux 环境的 home 目录 `~`。WSL 也可以在开始菜单中搜索找到并启动。也可以在 PowerShell 中键入

```ansi
> [35mwsl[0m -d archlinux
```

在 Windows 的当前目录下启动 Linux 环境。使用 `exit` 关闭终端，即可退出 WSL。
:::

## Linux

### 仓库

先同步软件仓库

```bash
pacman -Syu
```

安装必要软件：

```bash
pacman -S vim sudo which fastfetch btop
```

默认情况下，`pacman` 只有 `[core]` 和 `[extra]` 两种仓库。我可以定义 32 位仓库 `[multilib]` 以及 Arch Linux 中文社区仓库 `[archlinuxcn]`，修改 `/etc/pacman.conf`：

```text title="/etc/pacman.conf" ins={"32 位仓库":18-23} ins={"中文社区仓库":31-33}
# The testing repositories are disabled by default. To enable, uncomment the
# repo name header and Include lines. You can add preferred servers immediately
# after the header, and they will be used before the default mirrors.

#[core-testing]
#Include = /etc/pacman.d/mirrorlist

[core]
Include = /etc/pacman.d/mirrorlist

#[extra-testing]
#Include = /etc/pacman.d/mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist

# 32 bit applications

#[multilib-testing]
#Include = /etc/pacman.d/mirrorlist

[multilib]
Include = /etc/pacman.d/mirrorlist

# An example of a custom package repository.  See the pacman manpage for
# tips on creating your own repositories.
#[custom]
#SigLevel = Optional TrustAll
#Server = file:///home/custompkgs

[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

配置好 `/etc/pacman.conf` 后，`pacman` 会从仓库指定的镜像源 (比如 `[core]`、`[extra]` 对应的 `/etc/pacman.d/mirrorlist`) 下载软件。

因为一些原因，我需要为仓库准备中国相应的**镜像源**，这可以通过 Arch Linux 官网提供的 [Pacman Mirrorlist Generator](https://archlinux.org/mirrorlist/) 获取。

修改 `/etc/pacman.d/mirrorlist`，更换镜像源

```bash
vim /etc/pacman.d/mirrorlist
```

```text title="/etc/pacman.d/mirrorlist"
# Server = https://fastly.mirror.pkgbuild.com/$repo/os/$arch
# Server = https://geo.mirror.pkgbuild.com/$repo/os/$arch

## China
Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch
Server = https://mirrors.aliyun.com/archlinux/$repo/os/$arch
Server = http://mirrors.nju.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.nju.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
```

更换完后，建议再次同步新加的仓库

```bash
pacman -Syu
```

### root 用户

我给 `root` 用户配置默认编辑器为 `vim`：

```bash
vim ~/.bash_profile
```

```text title="~/.bash_profile" ins={1}
export EDITOR='vim'
```

### 非 root 用户

#### sudo

我为本机添加一位名为 `nisemono`、可通过 `sudo` 提权的用户：

```bash
useradd -m -G wheel -s /bin/bash nisemono
passwd nisemono
```

谁能 `sudo`？`sudo` 之后他的权限如何？我需要管理他们。

我使用 `vim` 编辑器 (环境变量 `EDITOR` 尚未生效)，通过 `visudo` 命令编辑 `sudoers` 文件，规定处于 `wheel` 用户组里的所有用户 (包括 `nisemono`) `sudo` 后的权限：

```bash
EDITOR=vim visudo
```

```text title="sudoers" del={1}, ins={2}
#%wheel ALL=(ALL:ALL) ALL
%wheel ALL=(ALL:ALL) ALL
```

:::tip

> 默认管理组的命名不统一：Debian 系 (Debian、Ubuntu、Linux Mint) 是 `sudo`，而 Red Hat 系 (RHEL、CentOS、Fedora) 和传统的 Unix (如 BSD) 是 `wheel`
>
> 可以通过查看 `/etc/sudoers` 确认
>
> ```text
> %sudo ALL=(ALL:ALL) ALL
> %wheel ALL=(ALL:ALL) ALL
> ```

`/etc/sudoers` 的配置示例：

```text
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

```text
# 别名
User_Alias      WEBADMINS = alice, bob, %devops
Cmnd_Alias      WEB_CMDS = /usr/bin/systemctl restart nginx, /usr/bin/systemctl reload nginx
Runas_Alias     WEBUSER = www-data

# 授权部分
WEBADMINS ALL=(WEBUSER) WEB_CMDS
```

:::

#### WSL 默认启动用户

现在我希望每次打开 WSL，登录的是 `nisemono` 用户，而不是 `root` 用户，因此修改 `/etc/wsl.conf`：

```text title="/etc/wsl.conf" ins={4-5}
[boot]
systemd=true

[user]
default=nisemono
```

彻底关闭 WSL

```ansi
> [35mwsl[0m --shutdown
```

重启 WSL，登录 `nisemono`。

#### yay

为 `nisemono` 获取 `[archlinuxcn]` 仓库的签名后，安装 `archlinuxcn/yay`，它可以让我安装 AUR 中的软件：

```bash
sudo pacman -S archlinuxcn-keyring
sudo pacman -S yay
```

#### 默认文本编辑器

让 `vim` 作为 `nisemono` 的默认文本编辑器：

```text title="~/.bashrc" ins={12}
#
# ~/.bashrc
#

# If not running interactively, don't do anything
[[ $- != *i* ]] && return

alias ls='ls --color=auto'
alias grep='grep --color=auto'
PS1='[\u@\h \W]\$ '

export EDITOR='vim'
```

#### fish

为 `nisemono` 安装 `fish`：

```bash
sudo pacman -S fish
which fish
```

:::tip
`fish` 全称 Friendly Interactive SHell，有许多开箱即用的功能，如自动补全、语法高亮、命令简写等。相比 `bash` 的兼容性、`zsh` 的高度可定制，`fish` 更易上手。
:::

启动 `fish`，执行 `fish_config` 命令后，可以非常方便地在本机 `8000` 端口自由设置 `fish` 的配色、prompt、函数、变量：

```ansi
$ fish

Welcome to fish, the friendly interactive shell
Type [32mhelp[0m for instructions on how to use fish
[32mnisemono[0m@shy-pc-windows ~> fish_config
...
[32m~[0m $ 
```

| 快捷键 | 功能 |
| ----- | --- |
| `Alt + Left` | 在目录历史中后退一步 ⭐ |
| `Alt + Right` | 在目录历史中前进一步 |
| 命令 `cdh` | 列出目录历史，字母/数字选择 ⭐⭐ |
| `Tab` | 路径补全 |
| `Right` | 接受命令补全 |
| `Ctrl + R` | 列出相关命令，方向键选择 ⭐ |
| `Ctrl + C` | 清空当前输入的命令 |
| `Ctrl + A` | 将光标移至命令开头 ⭐⭐ |
| `Ctrl + E` | 将光标移至命令末尾 |
| `Alt + F/B` | 按单词移动光标 ⭐⭐ |
| `Ctrl + W` | 删除光标前的一个单词 ⭐⭐ |
| `Alt + E` | 使用 `EDITOR` 编辑命令 |
| `Ctrl + Z` | 撤销误删操作 |
| `Alt + /` | 重做撤销的操作 |

#### Node.js

退出 `fish` 回到 `bash`，安装参考 [Node.js](https://nodejs.org/zh-cn/download)

```bash
[arishimu@arishi-laptop ~]$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
[arishimu@arishi-laptop ~]$ \. "$HOME/.nvm/nvm.sh"
[arishimu@arishi-laptop ~]$ nvm install 24
[arishimu@arishi-laptop ~]$ node -v
v24.18.0
[arishimu@arishi-laptop ~]$ which node
/home/arishimu/.nvm/versions/node/v24.18.0/bin/node
[arishimu@arishi-laptop ~]$ which npm
/home/arishimu/.nvm/versions/node/v24.18.0/bin/npm
[arishimu@arishi-laptop ~]$ corepack enable pnpm
[arishimu@arishi-laptop ~]$ pnpm -v
11.13.1
[arishimu@arishi-laptop ~]$ which pnpm
/home/arishimu/.nvm/versions/node/v24.18.0/bin/pnpm
```

:::warning

如果使用 `pacman` 安装 Node.js

```bash
arishimu@arishi-laptop ~> sudo pacman -S nodejs npm
arishimu@arishi-laptop ~> which node
/usr/sbin/node
arishimu@arishi-laptop ~> which npm
/mnt/c/Program Files/nodejs/npm
```

为了避免这种尴尬的情况，我们需要禁止 Windows 的 PATH 并入 WSL

```text title="/etc/wsl.conf" ins={7-8}
[boot]
systemd=true

[user]
default=nisemono

[interop]
appendWindowsPath=false
```

:::

### 代理

默认的 NAT 网络模式与宿主机网络隔离，导致无法自动使用宿主机的代理

```text
wsl: 检测到 localhost 代理配置，但未镜像到 WSL。NAT 模式下的 WSL 不支持 localhost 代理。
```

为了让 WSL 与 Windows 共享网络，自动继承主机的代理配置，在 Windows 的用户文件夹创建

```text title="C:\Users\ABCD\.wslconfig"
[experimental]
networkingMode=mirrored
autoProxy=true
```

彻底关闭 WSL 后重启

### Niri

:::caution
经实践，在 WSL 下使用 Niri 体验十分不佳：

1. WSLg 主要支持运行 Linux GUI 应用，而不提供完整桌面环境支持；
2. 当你的计算机屏幕分辨率达到或超过 2k 时，`niri-session` 无法弹出窗口。我尝试过手动降低屏幕分辨率，然后打开终端 `niri msg outputs` 获取信息，然后在 `~/.config/niri/config.kdl` 里修改配置，但最终窗口十分不稳定，非常容易闪退；
3. 弹出的窗口无法真正全屏。

因此下面有关 `Niri` 的内容都是废弃的。
:::

接下来我将安装 Wayland 窗口管理器 (Compositor) [**Niri**](https://github.com/niri-wm/niri)，支持可滚动平铺 (scrollable-tiling) 窗口。

:::note
与 Sway 或 Hyprland 不同的地方在于，Niri 将窗口排列在一个无限延伸的水平桌面上，你可以向左或向右滚动（当然也可以实现更复杂的布局）。
:::

安装桌面常用工具，比如 Niri 默认使用的**应用启动器** `fuzzel` 和**终端** `alacritty`。为了兼容不支持 Wayland 显示协议的图形程序，我还要安装 `xdg-desktop-portal-gtk` 和 `xdg-desktop-portal-gnome`。[Arch Wiki](https://wiki.archlinuxcn.org/zh/Niri) 推荐了这些软件包：

```bash
pacman -S fuzzel mako waybar xdg-desktop-portal-gtk xdg-desktop-portal-gnome alacritty swaybg swayidle swaylock xwayland-satellite udiskie
```

安装完软件后，我可以直接启动 Niri：

```bash
niri-session
```

<!-- code, tmux, docker -->