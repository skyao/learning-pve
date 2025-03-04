---
title: "带管理网络的 SR-IOV 实战"
linkTitle: "带管理网络"
weight: 10
date: 2021-01-18
description: >
  在开启 SR-IOV 给虚拟机共享网卡的同时支持管理网络
---



## 背景

需求：

- 在 pve 的各个虚拟机中分享网卡
- 同时支持管理网络也使用这个网卡， 从而实现管理网络的高速通讯

设备：

- 一块 40G/56G 网卡，型号为 HP544+，双口（但当前实战中只使用到一个口）

目标：

- 虚拟机中可以自由使用该网卡实现高速通讯
- 管理网络也使用这个网卡

规划：

-  HP544+ 网卡
  - 开启 SR-IOV
  - PF 用于 vmbr0 实现管理网络
  - VF 直通给到虚拟机


实现步骤：

1. 先安装 PVE, 使用 HP544+ 网卡做管理网络，vmbr0 会基于 HP544+ 网卡
2. 开启直通，设置 HP544+ 网卡 的 SR-IOV 配置
4. 配置  ubuntu server 虚拟机，直通 VF
5. 在 ubuntu server 虚拟机中检验网卡的使用

## 准备工作

### 安装 PVE

正常安装 PVE, 安装时网卡选择 HP544+ 网卡，填写网络信息类似为：

- ip 地址： 192.168.99.18
- 网关：192.168.99.1
- DNS： 192.168.99.1

PVE 安装完成后，检查是否可以连接到网关

```bash
ping 192.168.99.1
```

如果有问题，则需要

```bash
ip addr
```

查看网卡情况，然后执行

```bash
vi /etc/network/interfaces
```

修改 vmbr0 的配置，指向正确的网卡。然后执行

```bash
ifup -a
```

让网卡配置生效。重新 ping 来检验连接是否成功。

### 安装网卡相关软件

需要安装以下工具软件：

- mft 工具
- mlxup
- mstflint

## 实战步骤

### 开启直通

按照前面介绍的方式，开启 pve 的直通支持。

### 设置 SR-IOV

修改 HP544+ 网卡配置：

```bash
mst start

mlxconfig -d /dev/mst/mt4103_pci_cr0 reset

mlxconfig -d /dev/mst/mt4103_pciconf0 set SRIOV_EN=1 NUM_OF_VFS=8 WOL_MAGIC_EN_P2=0 LINK_TYPE_P1=2 LINK_TYPE_P2=2
```

然后执行

```bash
vi /etc/modprobe.d/mlx4_core.conf
```

修改网卡配置为只在 port 1 上建立8个 vf：

```properties
options mlx4_core port_type_array=2,2 num_vfs=8,0,0 probe_vf=8,0,0 log_num_mgm_entry_size=-1
```

最后执行

```bash
update-initramfs -u
```

重启机器。

此时 ip addr 看到的是：

```bash
4: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master vmbr0 state UP group default qlen 1000
    link/ether 48:0f:cf:ef:08:11 brd ff:ff:ff:ff:ff:ff
    altname enp129s0
5: ens4d1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 48:0f:cf:ef:08:12 brd ff:ff:ff:ff:ff:ff
    altname enp129s0d1
6: ens4v0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 7e:e8:bb:78:9b:e4 brd ff:ff:ff:ff:ff:ff
    altname enp129s0v0
7: ens4v1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 52:59:cf:53:10:ac brd ff:ff:ff:ff:ff:ff
    altname enp129s0v1
8: ens4v2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 02:d6:04:5b:7e:63 brd ff:ff:ff:ff:ff:ff
    altname enp129s0v2
9: ens4v3: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether ee:89:43:2e:af:d3 brd ff:ff:ff:ff:ff:ff
    altname enp129s0v3
10: ens4v4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 8e:8d:ac:50:63:17 brd ff:ff:ff:ff:ff:ff
    altname enp129s0v4
11: ens4v5: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 7e:b7:b5:c7:2a:bf brd ff:ff:ff:ff:ff:ff
    altname enp129s0v5
12: ens4v6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 76:47:57:f5:b7:c8 brd ff:ff:ff:ff:ff:ff
    altname enp129s0v6
13: ens4v7: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 3e:10:37:3c:86:49 brd ff:ff:ff:ff:ff:ff
    altname enp129s0v7
14: ens4v8: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 76:34:75:33:95:6b brd ff:ff:ff:ff:ff:ff
    altname enp129s0v8
15: ens4v9: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether ee:13:54:87:51:bc brd ff:ff:ff:ff:ff:ff
    altname enp129s0v9
16: ens4v10: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 5a:33:72:74:5a:78 brd ff:ff:ff:ff:ff:ff
    altname enp129s0v10
17: ens4v11: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether ae:a4:58:ec:ee:0a brd ff:ff:ff:ff:ff:ff
    altname enp129s0v11
18: vmbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 48:0f:cf:ef:08:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.99.58/24 scope global vmbr0
       valid_lft forever preferred_lft forever
```

执行

```bash
lspci | grep Mel
```

可以看到 

```bash
81:00.0 Ethernet controller: Mellanox Technologies MT27520 Family [ConnectX-3 Pro]
81:00.1 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
81:00.2 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
81:00.3 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
81:00.4 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
81:00.5 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
81:00.6 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
81:00.7 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
81:01.0 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
81:01.1 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
81:01.2 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
81:01.3 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
81:01.4 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
```



## 速度对比

### 虚拟机使用 vmbr0 

```bash
iperf -c 192.168.99.1 -P 10 -t 20 -i 1
```

实际测试下来大概是 20G：

```bash
[SUM]  0.0-20.1 sec  46.8 GBytes  20.0 Gbits/sec
```

### 虚拟机使用 VF

直通一个 VF 进入虚拟机，删除 vmbr0, 同样测速，结果大概是 42.8 G：

```bash
[SUM]  0.0-20.0 sec  99.8 GBytes  42.8 Gbits/sec
```

### 虚拟机直通网卡

这个之前测试过，速度可以达到 48G ，基本等同于物理机上测试的结果。

## 总结

以 HP544+ 网卡为例，开启 56G eth 模式，各种情况下的 iperf 测试结果是这样的：

1. 物理机下速度最快： 48G，得益于 rdma, CPU 占用基本为零
2. 虚拟机网卡直通基本等同于物理机：同样 48g 速度 + CPU 占用基本为零
3. PVE 系统直接使用 vmbr, 接近物理机和直通（算九五折）： 45到47.9g不等，cpu占用约 20%
4. 开启 SR-IOV 直通 VF, 速度下降：大约 42.8 g, 和最高性能相比打八折
5. 虚拟机使用vmbr：速度最慢，约 20G, 和最高性能相比打四折，而且 cpu 占用很高

所以，建议在不同场景下酌情使用：

- vmbr 方法：最灵活，但性能最低（打四折），适用于无法或者不方便开启直通/SR-IOV 的场合
- 直通方案：性能最高，直逼物理机，但有独占网卡的限制
- SR-IOV 直通 VF 方案：兼顾性能和灵活度，八折性能换来可以在多个虚拟机中共享网卡 （HP544+ 网卡最多可以有 15 个 VF）
