---
title: "ifupdown2工具"
linkTitle: "ifupdown2"
weight: 30
date: 2021-01-18
description: >
  使用 ifupdown2 配置PVE网络
---

参考：

https://pve.proxmox.com/pve-docs/pve-admin-guide.html#sysadmin_network_configuration

使用推荐的 ifupdown2 软件包（自Proxmox VE 7.0以来的新安装默认），可以在不重新启动的情况下应用网络配置更改。如果通过 GUI 更改网络配置，则可以单击“应用配置”按钮。这会将更改从暂存接口.new 文件移动到 /etc/network/interfaces 并实时应用它们。

下面是自带的 ifupdown2 软件包的版本情况：

```bash
ifup --version
info: executing /usr/bin/dpkg -l ifupdown2
ifupdown2:3.2.0-1+pmx3

ifdown --version
info: executing /usr/bin/dpkg -l ifupdown2
ifupdown2:3.2.0-1+pmx3
```

