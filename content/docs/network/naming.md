---
title: "网络设备的命名约定"
linkTitle: "命名约定"
weight: 20
date: 2021-01-18
description: >
  PVE网络中网络设备的命名约定
---

参考：

https://pve.proxmox.com/pve-docs/pve-admin-guide.html#sysadmin_network_configuration

3.3.2. Naming Conventions 一节。

## 命名约定

目前对设备名称使用以下命名约定：

- 以太网设备：en*，systemd 网络接口名称。此命名方案用于自版本 5.0 以来的新 Proxmox VE 安装。
- 以太网设备：eth[N]，其中 0 ≤ N （eth0， eth1， ...）此命名方案用于在5.0版本之前安装的Proxmox VE主机。升级到 5.0 时，名称将保持原样。
- 网桥名称：vmbr[N]，其中 0 ≤ N ≤ 4094 （vmbr0 - vmbr4094）
- bond：bond[N]，其中 0 ≤ N （bond0， bond1， ...）
- VLAN：只需将 VLAN 编号添加到设备名称中，用句点分隔（eno1.50、bond1.30）

下面是一个默认安装后的 `/etc/network/interfaces` 文件内容：

```bash
auto lo
iface lo inet loopback

iface enp7s0 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.0.18/24
        gateway 192.168.0.1
        bridge-ports enp7s0
        bridge-stp off
        bridge-fd 0

iface enp179s0 inet manual

iface enp179s0d1 inet manual

iface enp101s0 inet manual

iface enp101s0d1 inet manual
```

### systemd 网络接口名称

Systemd 对以太网网络设备使用两个字符前缀 en。接下来的字符取决于设备驱动程序以及哪个架构首先匹配的事实。

- `o<index>[n<phys_port_name>|d<dev_port>]`  — 板载设备
- `s[<slot>f<function>][n<phys_port_name>|d<dev_port>]`  — 按热插拔 ID 划分的设备
- `[P<domain>]p<bus>s[<slot>f]<function>[n<phys_port_name>|d<dev_port>] ` — 按 bus id 划分的设备
- `x<MAC> ` — 按 MAC 地址划分的设备

最常见的模式是：

- eno1 — 是第一个板载网卡
- enp3s0f1 — 是 pcibus 3 插槽 0 上的网卡，并使用网卡功能 1。

有关详细信息，请参阅 [可预测的网络接口名称](https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/)。
