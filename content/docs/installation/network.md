---
title: "修改管理网络"
linkTitle: "管理网络"
weight: 80
date: 2023-07-09
description: >
  在安装完成后修改管理网络
---

管理网络的设置通常在安装时完成，但有时需要在安装完成之后修改管理网络。

### 通过web页面

如果还能打开 web 页面，则非常简单，在节点的 "System" -> "Network" 中找到管理网络（通常是一个 linux bridge），“Edit” 就可以修改。

这个方案适合修改 ip 地址，网管等基础信息。也可以修改 Bridge Ports 信息来指定其他网卡作为管理网路。

### 通过命令行

如果遇到 web 页面已经无法打开，比如更换网卡之后，网卡名称发生变化，作为管理网络的 linux bridge 会因为底层网卡不可用导致  linux bridge 也不可用。

这种情况下只能通过命令行来操作，登录之后，

```bash
vi /etc/network/interfaces
```

打开文件，修改各个 iface 对应的网卡信息，对于管理网络，需要修改 linux bridge 的 address / gateway / bridge-ports 等信息：

```properties
auto lo
iface lo inet loopback

iface enp11s0 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.0.18/24
        gateway 192.168.0.1
        bridge-ports enp11s0
        bridge-stp off
        bridge-fd 0

iface enp179s0 inet manual

iface enp179s0d1 inet manual
```

### 修改 hosts 文件

记得修改 hosts 文件中的记录：

```bash
vi /etc/hosts
```

这里也有一个旧的 IP 地址需要为新的 IP 地址。

修改完成后重启。