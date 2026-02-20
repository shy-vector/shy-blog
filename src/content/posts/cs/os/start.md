---
title: OS 随记
published: 2026-02-19
updated: 2026-02-19
description: '一些零散的操作系统知识'
image: ''
tags: ['OS']
category: 'OS'
draft: false
---

# OS 随记

:::warning
以下内容不一定准确，存在大量简化模型．
:::

## 杂记

### 开机后发生了什么？

:::note[固件 (Firmware)]
固件是一段被烧录在主板 ROM 上的程序，用于初始化硬件，引导操作系统的启动．

常见的固件有 BIOS (Basic I/O System) 和 UEFI (Unified Extensible Firmware Interface，统一可扩展固件接口) 固件 (由各家主板厂商按照 UEFI 规范实现)． 
:::

电路逻辑：

1. 电源被接通．
2. 时钟发生器产生时钟脉冲．
3. 所有寄存器被复位．
4. 设置 PC 地址 (对于 x86 架构的 CPU 是 `0xFFFFFFF0`)，这个地址由 PCH (主板芯片组) 硬编码至主板的 ROM 上．存放的内容是计算机即将执行的第一条指令：跳转指令 (如 `jmp F000:FFF0` 或 `jmp F000:E05B`)，跳转地址是固件的起始位置．
5. 开启指令周期 (取指，解码，执行)．此时 CPU 在 16 位实模式下运行，DRAM 尚未初始化．

固件运行：

1. POST (Power On Self Test，电源自检)：检查关键硬件是否正常工作，对应主板上亮着的 Debug 灯．
2. SEC + PEI 阶段：使用 Cache 当作临时内存，初始化内存控制器，让内存可用．
<!-- 3. 在 RAM 上建立 IVT (中断向量表，每个向量指向 ROM 里相应的 ISR (中断服务例程) 入口地址) (CPU 实模式，没有内存保护) 或 IDT (中断描述符表，利用门描述符间接跳转) (CPU 保护模式)． -->
3. DXE 阶段：加载驱动程序 (PCI、USB、SATA、Console)，构建 Boot Services (gBS，启动服务，负责内存管理、驱动加载、控制台输出等) 和 Runtime Services (gRT，运行时服务，负责维护系统时间、变量存储、系统重启等)．在这之后，硬盘、显卡、键盘等设备可以开始工作了．
4. BDS 阶段：寻找操作系统．UEFI 读取 NVRAM (非易失性内存) 中的 Boot Entries (硬盘的启动顺序表) ：
   ```text
   1. [UEFI: Samsung SSD 970 EVO Plus] -> m.2 固态硬盘 (启动盘)
   2. [UEFI: Kingston SSD A400]        -> SATA 固态硬盘
   3. [UEFI: USB Flash Drive]          -> U 盘
   4. [Legacy: WDC HDD]                -> 机械硬盘
   5. [Network: PXE]                   -> 网络启动
   ```
   按顺序依次查找 GPT 格式的磁盘，在 GPT 上寻找 ESP (EFI System Partition，EFI 系统分区)，并在 ESP 寻找 `.efi` 引导程序 (用于加载操作系统内核至内存，通常是 Boot Loader，如 Windows Boot Manager `\EFI\Microsoft\Boot\bootmgfw.efi`，GRUB `/boot/grubx64.efi`)．
5. TSL 阶段：引导程序开始执行，加载操作系统．以 GRUB 为例，先加载配置文件 `/boot/grub/grub.cfg`，显示菜单界面，用户选择操作系统，选择完毕后加载操作系统内核至内存．
6. RT 阶段：操作系统完全接管计算机，此时 Boot Services 不可用 (内存管理、硬件访问等功能 UEFI 不再负责，转由操作系统负责)，Runtime Services 继续正常工作．(如果是 Windows，这时屏幕中央开始显示徽标)

:::tip[GPT 格式]
相比 MBR (Master Boot Record, 1983, IBM PC DOS 2.0) 格式，GPT (Globally Unique Identifier Partition Table, GUID Partition Table，全局唯一标识分区) 格式有许多优势：

| 格式 | 最大分区容量 | 最大分区数 | 系统 |
| --- | --- | --- | --- |
| MBR | 2TB | 4 | Windows 7 及以下 |
| GPT | 9.4ZB | 128 | Windows 8 及以上 |

GPT 表项格式：

| 起始字节 | 内容 |
| --- | --- |
| 0 | 分区类型 |
| 16 | 分区 GUID |
| 32 | 起始 LBA (小端序) |
| 40 | 末尾 LBA |
| 48 | 属性 (如 `60` 表示只读) |
| 56 | 分区名 (72 字节) |

GPT 格式的硬盘 (如 `/dev/sda`、`/dev/nvme0n1`) 分区类似于

| 分区 | 容量 | 文件系统 |
| --- | --- | --- |
| ESP | ≈ 500 MB | FAT32 |
| Windows | 500+ GB | NTFS |
| Arch Linux | 300+ GB | Btrfs |
| Swap | ≈ 80 GB | - |
:::

### 使用相同指令集的程序，为什么不能跨平台运行？

### 上下文如何切换？
