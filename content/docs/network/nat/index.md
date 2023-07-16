---
title: "NAT"
linkTitle: "NAT"
weight: 120
date: 2021-01-18
description: >
  PVE网络中的NAT
---

参考：

https://pve.proxmox.com/pve-docs/pve-admin-guide.html#sysadmin_network_configuration

3.3.6. Masquerading (NAT) with iptables 一节。

## NAT

伪装(Masquerading)允许只有专用 IP 地址的 guest 通过将主机 IP 地址用于传出流量来访问网络。iptables 会重写每个传出数据包，使其显示为源自主机，并相应地重写响应以路由到原始发送方。

安装程序会创建一个名为 vmbr0 的单个网桥，该网桥连接到第一个以太网卡。/etc/network/interfaces 中的相应配置可能如下所示：

```bash
auto lo
iface lo inet loopback

auto eno1
#real IP address
iface eno1 inet static
        address  198.51.100.5/24
        gateway  198.51.100.1

auto vmbr0
#private sub network
iface vmbr0 inet static
        address  10.10.10.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0

        post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up   iptables -t nat -A POSTROUTING -s '10.10.10.0/24' -o eno1 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.10.10.0/24' -o eno1 -j MASQUERADE
```

