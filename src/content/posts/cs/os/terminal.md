---
title: 终端，虚拟终端，终端模拟器，伪终端，Shell
published: 2026-07-25
updated: 2026-07-25
description: '用户视角下的 TTY/PTY'
image: ''
tags: ['OS', 'Linux']
category: 'OS'
draft: false
---

:::note

1. 裸机：ENIAC
   - 在硬件上构建计算流程
   - 在氖灯或电压表体现输出
2. 批处理操作系统 (1950s)：程序逻辑脱离硬件，可存放在内存中
   - 程序员在纸卡上打孔编写程序，准备祭品
   - 一摞纸卡组成作业，程序员作业将交给操作员
   - 操作员进行作业管理，选择作业并放入读卡器
   - 计算机按顺序读取纸卡上的程序
   - 计算机执行程序，在打印纸里降下神谕
   - 程序员取出运行结果/报错信息，获得圣旨
3. 分时操作系统 (1960s - )：主机支持多任务，因此可以和用户实时交互
   - 主机同时连接多台终端，用户使用终端与主机交互
   - 主机轮流服务多台终端，用户有独占机器的错觉
   - **终端 ≈ 键盘 + 输出设备**，所有终端都包装了键盘用于输入
   - 电传打字机 (Teletype, tty) 在打印纸上体现输出
   - 视频终端 (Video Terminal) 在屏幕上体现输出，屏幕支持闪烁光标、彩色输出 (1970s, Unix 诞生)
   - 程序 (如 `bash`, `top`, `vim`) 的输出不应依赖多样的终端类型，因此需要抽象出统一接口，**虚拟终端** (今天的 **tty**) 孕育而生，它在软件上模拟视频终端

:::

## 启动

你在引导启动搭载 Linux 内核的计算机时，启动参数 `console=...` 会具体指定控制台 `/dev/console` (系统级别的，直达内核，属于机房操作员的面板)，默认是当前活动的虚拟终端 (`tty0`)，也可指定串口 (如 `ttyS0`)，它承载了内核所有 `printk()` 的输出信息 (如开屏的一串启动信息，刷屏后才进入登录界面)。

内核的虚拟终端子系统 (VT subsystem) 为你准备了多台虚拟终端 (字符设备 `/dev/tty1`, `/dev/tty2`, ..., `/dev/tty6`)，在启动时 `systemd` 为它们运行 `agetty` 进程 (`ubuntu login:`)。得益于 VT 子系统的多路复用功能，个人计算机的这些 tty 承载着同一套键盘和显示器，使用 `[Ctrl + Alt + Fx]` 切换 tty。

:::tip

打开文件 `/etc/systemd/logind.conf` 并将选项 `NAutoVTs=6` 设置为你希望在启动时拥有的虚拟终端数量。

:::

:::tip

想要用两套显示器 (显卡)、键盘，可以使用 Multiseat 功能：用户通过 `loginctl` 为特定设备绑定到特定 seat 上，系统会为每个 seat 对应的 tty 启动独立的显示管理器 (X / Wayland)。两套物理外设就像两台独立的电脑，各自有登录界面和会话，各自可以使用 `[Ctrl + Alt + Fx]` 切换 tty，彼此完全隔离。

:::

## 登录

当你在 `tty2` 上键入用户名时，依次按下 `Box[Backspace]b[Enter]`：

1. PS/2 (AT) 键盘产生硬件中断
2. 系统的 USB 驱动程序 [atkbd](https://docs.kernel.org/input/input.html#atkbd) 进行中断处理，将中断转成标准的 [输入事件](https://docs.kernel.org/input/input.html#event-interface) (比如 `KEY_SHIFT` + `KEY_B`，还可以用于封装鼠标、触控屏、游戏手柄的事件)

   ```cpp
   /* include/linux/input.h */
   struct input_value {
     __u16 type; // type of value (EV_KEY, EV_ABS, etc)
     __u16 code;
     __s32 value;
   };
   ```

   递交至 [**输入子系统 (Input Subsystem)**](https://docs.kernel.org/driver-api/input.html#input-subsystem) 的核心
3. 输入子系统完成了对输入 handler 及其感兴趣的输入设备的抽象

   ```cpp
   /* include/linux/input.h */
   struct input_dev {
     const char *name;
     const char *phys;
     const char *uniq;
     struct input_id id;
     ...
   };

   struct input_handler {
     void *private;
     void (*event)(struct input_handle *handle, unsigned int type, unsigned int code, int value);
     unsigned int (*events)(struct input_handle *handle, struct input_value *vals, unsigned int count);
     bool (*filter)(struct input_handle *handle, unsigned int type, unsigned int code, int value);
     bool (*match)(struct input_handler *handler, struct input_dev *dev);
     int (*connect)(struct input_handler *handler, struct input_dev *dev, const struct input_device_id *id);
     void (*disconnect)(struct input_handle *handle);
     void (*start)(struct input_handle *handle);
     bool passive_observer;
     bool legacy_minors;
     int minor;
     const char *name;
     const struct input_device_id *id_table;
     struct list_head        h_list;
     struct list_head        node;
   };
   ```

   输入子系统核心收到驱动的 `input_event` 后，分发给所有注册过且对该设备类型感兴趣的 `input_handler`。
4. 常见的 `input_handler` 有：
   - `evdev_handler`：通用用户态接口。任何用户态程序 (Xorg、Wayland、evtest) 都可以用标准文件操作对 `/dev/input/eventN` 直接读取原始的 `input_event` 流 (保存在内核的 buffer)，是图形桌面、触摸屏手势、游戏手柄的标准通道
   - `kbd_handler` (当前场景)：虚拟终端键盘。专门把键盘事件递交给 **虚拟终端子系统 (VT subsystem)**。当图形服务器通过 `evdev` 独占键盘时，`kbd_handler` 会被抑制。
5. VT 子系统
   - 数据结构：终端颜色、键盘布局 (keymap)、光标位置等
   - **屏幕绘制**：按照编译在内核里的字体位图，将字符图形绘至帧缓冲设备(fbcon 模式) 或显存 (VGA 文本模式)
   - 多路复用：活动控制台的选择受用户的 `[Ctrl + Alt + Fx]` 控制，通过设备文件 `/dev/ttyN` 向用户态暴露

   ```cpp
   /* include/linux/console_struct.h */
   struct vc_font {
     unsigned int width;
     unsigned int height;
     unsigned int charcount;
     const unsigned char *data;
   };

   unsigned int vc_font_pitch(const struct vc_font *font);
   unsigned int vc_font_size(const struct vc_font *font);

   struct vc_data {
     struct tty_port port;   /* Upper level data */

     struct vc_state state, saved_state;

     unsigned short vc_num;   /* Console number */
     unsigned int vc_cols;  /* [#] Console size */
     unsigned int vc_rows;
     unsigned int vc_size_row;  /* Bytes per row */
     unsigned int vc_scan_lines;  /* # of scan lines */
     unsigned int vc_cell_height;  /* CRTC character cell height */
     unsigned long vc_origin;  /* [!] Start of real screen */
     unsigned long vc_scr_end;  /* [!] End of real screen */
     unsigned long vc_visible_origin; /* [!] Top of visible window */
     unsigned int vc_top, vc_bottom; /* Scrolling region */
     const struct consw *vc_sw;
     unsigned short *vc_screenbuf;  /* In-memory character/attribute buffer */
     unsigned int vc_screenbuf_size;
     unsigned char vc_mode;  /* KD_TEXT, ... */
     /* cursor */
     unsigned int vc_cursor_type;
     unsigned long vc_pos;   /* Cursor address */
     /* fonts */
     unsigned short vc_hi_font_mask; /* [#] Attribute set for upper 256 chars of font or 0 if not supported */
     struct vc_font vc_font;   /* Current VC font set */
     ...
     struct vt_mode vt_mode;
     struct pid  *vt_pid;
     int  vt_newvt;
     wait_queue_head_t paste_wait;
     /* mode flags */
     ...
   };

   struct vc {
     struct vc_data *d;
     struct work_struct SAK_work;

     /* might add  scrmem, kbd  at some time,
       to have everything in one place */
   };
   ```

   接收来自 `kbd_handler` 的 `input_event`，**根据当前 keymap 将 `input_value` 转换为 `char`** (`[Shift + B]` 转成字符 `B`)，然后注入到该控制台对应的 tty 输入队列 (`struct tty_port port;`) ，进入 [**TTY 子系统 (TTY Subsystem)**](https://docs.kernel.org/driver-api/tty/index.html)
6. TTY 子系统

   [浅析 TTY Subsystem](https://blog.sjtuxhw.top/technical/tty-subsystem/index.html)

   **TTY 设备 (TTY Device)** 是三元组，对外以 `/dev/ttyN` 呈现：

   - 底层硬件驱动（在物理串口上是 UART 驱动，在虚拟终端上是 VT）
   - 线路规程 (Line Discipline)：一段代码，提供行编辑功能
   - TTY 驱动 (TTY Driver)：一段代码，提供会话管理

   ```cpp
   /* inlcude/linux/tty.h */
   struct tty_struct {
     struct kref kref;
     int index;
     struct device *dev;
     struct tty_driver *driver;
     struct tty_port *port;
     const struct tty_operations *ops;

     struct tty_ldisc *ldisc;
     struct ld_semaphore ldisc_sem;

     struct ktermios termios, termios_locked;
     char name[64];
     unsigned long flags;
     int count;
     unsigned int receive_room;
     struct winsize winsize;

     struct {
       spinlock_t lock;
       bool stopped;
       bool tco_stopped;
     } flow;

     struct {
       struct pid *pgrp;
       struct pid *session;
       spinlock_t lock;
       unsigned char pktstatus;
       bool packet;
     } ctrl;

     struct tty_struct *link;
     struct fasync_struct *fasync;
     wait_queue_head_t write_wait;
     wait_queue_head_t read_wait;
     struct work_struct hangup_work;
     void *disc_data;
     void *driver_data;
     spinlock_t files_lock;
     int write_cnt;
     u8 *write_buf;

     struct list_head tty_files;
     struct work_struct SAK_work;

     ...
   };

   /* include/linux/tty_buffer.h */
   struct tty_bufhead {
     struct tty_buffer *head; /* Queue head */
     struct workqueue_struct *flip_wq;
     struct work_struct work;
     struct mutex lock;
     atomic_t priority;
     struct tty_buffer sentinel;
     struct llist_head free; /* Free queue head */
     atomic_t mem_used; /* In-use buffers excluding free list */
     int mem_limit;
     struct tty_buffer *tail; /* Active buffer */
   };

   struct tty_buffer {
     union {
       struct tty_buffer *next;
       struct llist_node free;
     };
     unsigned int used;
     unsigned int size;
     unsigned int commit;
     unsigned int lookahead; /* Lazy update on recv, can become less than "read" */
     unsigned int read;
     bool flags;
     /* Data points here */
     u8 data[] __aligned(sizeof(unsigned long));
   };

   /* drivers/tty/n_tty.c */
   struct n_tty_data {
     /* producer-published */
     size_t read_head;
     size_t commit_head;
     size_t canon_head;
     size_t echo_head;
     size_t echo_commit;
     size_t echo_mark;
     DECLARE_BITMAP(char_map, 256);

     /* shared by producer and consumer */
     u8 read_buf[N_TTY_BUF_SIZE];
     DECLARE_BITMAP(read_flags, N_TTY_BUF_SIZE);
     u8 echo_buf[N_TTY_BUF_SIZE];

     /* consumer-published */
     size_t read_tail;
     size_t line_start;

     ...
   };
   ```

   - 数据结构：终端号 (`index`)，前台进程组 (`ctrl.pgrp`)，会话 `ctrl.session`，线路规程 (`ldisc`)，TTY 驱动 (`driver`) 等
   - [**线路规程 (line discipline)**](https://docs.kernel.org/driver-api/tty/tty_ldisc.html)：默认值为 `N_TTY`，工作在缓冲模式。进程可以通过 `termios` 接口切换模式
      - **缓冲模式** (`N_TTY`，当前场景)：
        - 回显：每收到一个字符，就写回输出队列，VT 层随后将其绘至屏幕
        - 行缓冲：字符暂存至 `n_tty_data.read_buf`，直至遇到 `\n` 或 `EOF`，才将 buffer 里面的字符串交到前台进程组
        - 特殊处理 (使用 `stty -a` 打印当前终端设置)：

          | 字符 | 行为 |
          | --- | --- |
          | `^H` (`[Backspace]`) | 删除 `read_buf` 末尾字符 |
          | `^U` (`[Ctrl + U]`) | 清空 `read_buf` |
          | `^C` (`[Ctrl + C]`) | 由 TTY Driver 向前台进程组发出 `SIGINT`，终止进程 |
          | `^\` (`[Ctrl + \]`) | 由 TTY Driver 向前台进程组发出 `SIGQUIT`，强制终止进程，并生成 core dump 文件用于调试 |
          | `^Z` (`[Ctrl + Z]`) | 由 TTY Driver 向前台进程组发出 `SIGTSTP`，挂起 (suspend) 进程至后台 |

      - 非缓冲模式 (`N_RAW`，raw mode)：字符逐个交付，不做行编辑和信号映射，`vim`、`top` 等全屏程序均使用此模式
   - 会话管理与作业控制：由 TTY Driver 负责，通过进程的系统调用被动触发
     - 每个 tty 都作为 **控制终端** 去关联一个 **会话 (session)** (以 `sid` 标识)，会话由一个前台进程组 (foreground) 和若干后台进程组 (background) 组成，仅有一个特殊进程作为 **会话首进程 (session leader)** (一般是 `setsid()` 创建这个 session 的进程，调用后脱离原有的控制终端，留下孤儿进程组)，每个新进程的 sid 与父进程相同
     - 用户往往同时有多个进程，这些进程的 `stdin`/`stdout`/`stderr` 都对准了同一个终端 (tty/pty)，但用户一次只能与其中一个交互
     - 系统调用 `tcsetpgrp()` 指定某个进程作为新的前台进程，并且记录在 `tty_struct.ctrl.pgrp`，调用 `waitpid()` 等待其结束或停止。而用户也可以通过 Shell 执行 `fg` 将进程组调回前台。这种轮流上台的机制就是 **作业控制 (Job Control)**
       - 键盘输入只被送往前台进程组
       - 线路规程收到 `^C` / `^Z` 时，向前台发送 `SIGINT` / `SIGTSTP` (比如 `kill_pgrp(tty->ctrl->pgrp, signal)`)
       - 后台进程试图从终端读取数据：被 `SIGTTIN` 挂起，直至 `fg`
       - 后台进程试图向终端写入数据：系统默许，除非配置终端 `stty tostop`，后台写操作被 `SIGTTOU` 挂起以避免多进程输出串扰
     - 如果控制终端的连接断开 (关闭终端模拟器窗口、拔掉串口、SSH 断连)，内核会向该终端的会话首进程 (通常是 shell) 发送 `SIGHUP`，会话首进程在退出前向自己的子进程组广播 `SIGHUP`，从而清理会话

```text
物理键盘
  │
  │  硬件中断
  ▼
USB/AT 驱动  →  input 子系统核心
  │
  │  kbd_handler 收到 KEY_B (值 48)
  ▼
VT 子系统 ─ 根据 keymap（如 us），Shift+B → 'B', Backspace → '^H'
  │
  │  将字符 'B' 加入 tty1 的输入队列
  ▼
tty 子系统 ─ 线路规程
  │
  ├→ 回显：输出 'B' 到 tty1 输出队列 → VT 绘制到屏幕
  ├→ 缓冲：'B' 暂存至行缓冲区
  │
  │  用户继续输入 'o','x', '^H', 'b'
  │  回车时，行缓冲完整提交 "Bob\n"
  ▼
/bin/login (前台进程) 的 read() 返回，获得字符串 "Bob\n"
```

认证完成后，`/bin/login` 将使用的终端 (当前场景是 `/dev/tty2`) 的所有者改为你登录的用户，并启动用户指定的 shell 程序。此后你和你的 shell 可以直接使用 `stty` 配置当前终端 `tty2`，不影响其他终端。

## 图形桌面

在现代主流 Linux 发行版 (如 Fedora，Ubuntu，Arch) 中，图形登录界面替代了黑底白字的文本提示，默认将图形界面放在 `tty1` 上。

**显示管理器 (DM)** 调用 `systemd-logind` 的接口，获得空闲的 VT，且通常为 `tty1`。`getty@tty1.service` 会因为与 `gdm/ssdm.service` 冲突而被显式停止或屏蔽。只有在 DM 崩溃退出时，`systemd` 才会重新激活 `tty1` 的 `agetty` 作为应急后备。

DM 启动 **显示服务器 (Xorg/Wayland Compositor)**，并通过启动参数让它接管与 `tty1` 关联的输入设备和显卡，然后通过 `ioctl` 调用 `KD_GRAPHICS` 模式，将 `tty1` 从文本模式切换到图形模式，绘制图形登录界面，等待用户认证。

:::note

此时，`tty1` 的文本帧缓冲被完全废弃，内核不再往该 `tty1` 写入字符，所有渲染交由显卡的 DRM 平面处理，所有键盘事件转由 `evdev_handler` 处理。进程直接从 `/dev/input/eventN` 读取输入事件，`kbd_handler` 被抑制，从而绕过 VT 子系统。

:::

认证完毕后，DM 启动用户选择的 **桌面环境** (如 KDE Plasma、GNOME 等)。

## 终端模拟器

在桌面环境中，用户可以使用 **终端模拟器 (Terminal Emulator)** (Terminal、Konsole、Xterm、iTerm2 等)，它们旨在图形桌面环境下，作为图形应用模拟虚拟终端环境 (而不必为了终端环境进入其他 tty)。

终端模拟器在启动后，通过 `openpty()` 打开 `/dev/ptmx` 向内核申请 **伪终端 (Pseudo Terminal, pty)** 对：

- master 端 (ptm)：由终端模拟器持有，作为在进入线路规程前的输入统一抽象
- slave 端 (pts)：对应设备文件 `/dev/pts/N`，模拟 TTY 设备

然后 `fork` 出子进程，让子进程的 `stdin`/`stdout`/`stderr` 对准 `/dev/pts/N` 之后再 `exec("/bin/bash")`，`bash` 从 `stdout` 输出

```bash
[Bob@laptop ~]$
```

:::note

此时 `bash` 有处在 tty 环境的错觉，执行 `tty` 后返回 `/dev/pts/N`。

:::

现在键入 `"python\n"`，与 tty 环境区别：

```text
键盘事件 (中断) → Input Subsystem (KEY_P) → Wayland/X11 服务器 (KEY_P) → 终端模拟器(字符 p) → ptm (字符 p) → [line discipline → pts] → /bin/bash
```

`bash` 解析 `"python\n"`，发现你要运行 `python3`，便 `fork` + `exec` 出 Python 进程。Python 进程继承了 `bash` 的文件描述符，因此 `stdin`/`stdout`/`stderr` 也对准 `/dev/pts/N`。

Python 进程通过 `stdout` 把 `>>>` 输出在 `/dev/pts/N` 上，显示在终端模拟器窗口里

```text
python3 → pts → ptm → 终端模拟器 → Wayland/X11 服务器 → 屏幕
```

当你按下 `[Ctrl + C]`，线路规程向前台 Python 进程发送 `SIGINT`，Python 收到后停止执行并输出 `KeyboardInterrupt`。

## SSH

服务器的 `sshd` 充当终端模拟器的角色持有 ptm，子进程持有 pts 负责与 shell 交互。这时原本用于屏幕绘制的 VT 子系统变成网络 I/O，传回客户端 `ssh`。

```text
客户端键盘 → Input Subsystem → Wayland/X11 服务器 → 终端模拟器 → 网络 → sshd → ptm → [line discipline → pts] → python3

python3 → pts → ptm → sshd → 网络 → 终端模拟器 → Wayland/X11 服务器 → 客户端屏幕
```
