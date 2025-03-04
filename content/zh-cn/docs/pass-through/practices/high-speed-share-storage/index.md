---
title: "高速共享存储"
linkTitle: "高速共享存储"
weight: 30
date: 2021-01-18
description: >
  开启直通来建立高速的共享存储
---



## 背景

需求：

- 希望在局域网内为其他机器提供高速网络共享服务

设备：

- 一块 40G/56G 网卡，型号为 HP544+，双口（但当前实战中只使用到一个口）
- 一块爱国者p7000z 4T pcie 4.0 硬盘
- 一块华硕z690-p d4 主板

目标：

- 虚拟机拥有 56G 高速网络，同时可以高速读写那块 4T 的 pcie4 ssd 硬盘

~~规划方案1~~：

-  HP544+ 网卡直通给到虚拟机
-  p7000z  硬盘直通给到虚拟机

规划方案2：

-  HP544+ 网卡直通给到虚拟机
-  p7000z 硬盘由 pve 管理，然后以 pve hard disk 的方式传递给虚拟机

## 遇到问题修改规划

规划1 在实践时遇到问题，HP544+ 网卡和 p7000z  硬盘都可以单独直通给到虚拟机，但是如果两个设备同时一起直通给到虚拟机，则会报错：

```bash
kvm: -device vfio-pci,host=0000:01:00.0,id=hostpci1,bus=ich9-pcie-port-2,addr=0x0: vfio 0000:01:00.0: Failed to set up TRIGGER eventfd signaling for interrupt INTX-0: VFIO_DEVICE_SET_IRQS failure: Device or resource busy
TASK ERROR: start failed: QEMU exited with code 1
```

google 一番没有找到解决方案，倒是找到了一个问题和我类似的帖子：

https://forum.proxmox.com/threads/passthrough-two-pci-devices.110397/

作者说问题发生在使用两块 Mellanox ConnectX-3 网卡时，当他换成 Mellanox ConnectX-4 网卡时问题就消失了。

考虑到目前我还没有换 Mellanox ConnectX-4 的计划，只能放弃规划方案1, 后面的操作是基于规划方案2。 

## 准备工作

### 准备 PVE 和直通支持

安装 pve 8.0, 开启直通支持。

### 准备虚拟机

安装 ubuntu server 20.04, 按照之前虚拟机的要求为直通做好准备。

## 实战步骤



## 速度对比

