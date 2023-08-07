---
title: "missing uar aborting"
linkTitle: "missing uar aborting"
weight: 10
date: 2021-01-18
description: >
  直通网卡时因驱动问题造成的 missing uar aborting 错误
---



## 故障描述

网卡直通给到虚拟机后，lspci 能看到设备：

```bash
lspci | grep Mel
01:00.0 Ethernet controller: Mellanox Technologies MT27520 Family [ConnectX-3 Pro]
02:00.0 Ethernet controller: Mellanox Technologies MT27520 Family [ConnectX-3 Pro]
```

```bash
lspci -k
```

可以看到网卡的驱动情况如下：

```bash
01:00.0 Ethernet controller: Mellanox Technologies MT27520 Family [ConnectX-3 Pro]
	Subsystem: Hewlett-Packard Company InfiniBand FDR/Ethernet 10Gb/40Gb 2-port 544+FLR-QSFP Adapter
	Kernel driver in use: vfio-pci
	Kernel modules: mlx4_core
02:00.0 Ethernet controller: Mellanox Technologies MT27520 Family [ConnectX-3 Pro]
	Subsystem: Hewlett-Packard Company InfiniBand FDR/Ethernet 10Gb/40Gb 2-port 544+FLR-QSFP Adapter
	Kernel driver in use: vfio-pci
	Kernel modules: mlx4_core
```

但是直通给虚拟机之后，在虚拟机中网卡无法识别，networkctl 看不到设备：

```bash
networkctl 
IDX LINK    TYPE     OPERATIONAL SETUP     
  1 lo      loopback carrier     unmanaged 
  2 enp6s18 ether    routable    configured

2 links listed.
```

lsmod 时发现只有 mlx4_core , 没有其他比如 mlx4_en：

```bash
lsmod  | grep mlx
mlx4_core             311296  0
```

对比正常情况下的 lsmod 输出应该是这样的：

```bash
lsmod | grep mlx   
mlx4_ib               245760  0
ib_uverbs             163840  1 mlx4_ib
mlx4_en               155648  0
ib_core               393216  2 mlx4_ib,ib_uverbs
mlx4_core             405504  2 mlx4_ib,mlx4_en
```

## 查找问题

在 pve 机器中同样执行 `lsmod | grep mlx` 命令，会发现存在和在虚拟机中同样的问题，输出中也只有 mlx4_core：

```bash
lsmod | grep mlx   
mlx4_core             462848  0
```

重新安装 ubuntu server 也无法解决这个问题。而且安装 ubuntu server 20.04 时，在最开始的启动界面，会看到类似这样的错误提示：

```bash
mlx4_core 0000:01:00.0: Missing UAR, aborting
```

在 pve 操作系统中执行命令：

```bash
dmesg | grep mlx    
```

也会看到类似的输出：

```bash
[    2.364990] mlx4_core: Mellanox ConnectX core driver v4.0-0
[    2.365005] mlx4_core: Initializing 0000:01:00.0
[    2.365098] mlx4_core 0000:01:00.0: Missing UAR, aborting
[    2.365214] mlx4_core: Initializing 0000:02:00.0
[    2.365309] mlx4_core 0000:02:00.0: Missing UAR, aborting
```

执行命令：

```bash
dmesg | grep mlx
```

输出类似为：

```bash
mlx_compat: loading out-of-tree module taints kernel.[0.991286] mlx_compat: module verification failed: signature and/or required key missing - tainting kernel[0.992456] mlx4_core: Mellanox ConnectX core driver v4.1-1.0.2 (27 Jun 2017)[0.992479] mlx4_core: Initializing 0000:01:00.0[0.992621] mlx4_core 0000:01:00.0: Missing UAR, aborting
```

基本可以判断存在某个问题导致网卡驱动无法正确加载，从而导致网卡无法识别和驱动。

## 修复方式

### 增加 "pci=realloc=off" 参数

google之后发现有人遇到类似的问题，验证可行：

https://forums.developer.nvidia.com/t/mlx4-core-missing-uar-aborting/207322/2

按照这里的意思，应该增加 `pci=realloc=off `：

- edit /etc/default/grub
- add GRUB_CMDLINE_LINUX_DEFAULT="pci=realloc=off"
- update-grub
- Reboot

### 修改主板 bios 设置，开启 ASPM 

在华硕z87-a主板，还需要在 bios 中打开 ASPM 相关的选项。

