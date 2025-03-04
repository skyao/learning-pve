---
title: "配置国内的更新源"
linkTitle: "更新源"
weight: 30
date: 2023-07-09
description: >
  配置国内的更新源
---

修改之前先备份一下：

```bash
cp /etc/apt/sources.list /etc/apt/sources.list.original
```

### 添加国内软件源

修改

```bash
vi /etc/apt/sources.list
```

内容为：

```properties
deb https://mirrors.ustc.edu.cn/debian/ bookworm main contrib
# deb-src https://mirrors.ustc.edu.cn/debian/ bookworm main contribe
deb https://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib
# deb-src https://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib
```

### 给 PVE 更换国内源

```bash
vi /etc/apt/sources.list.d/pve-no-subscription.list
```

内容为：

```properties
deb https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian bookworm pve-no-subscription
```

### 屏蔽 PVE 的企业源和 ceph 的源

先注释掉pve的企业源：

```bash
vi /etc/apt/sources.list.d/pve-enterprise.list
```

将内容注释即可：

```properties
#deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise
```

再注释掉 ceph 的源：

```bash
vi /etc/apt/sources.list.d/ceph.list
```

将内容注释即可：

```properties
#deb https://enterprise.proxmox.com/debian/ceph-quincy bookworm enterprise
```

执行命令：

### 更新

```bash
apt update
```

看一下有哪些可以更新的：

```bash
apt list --upgradable
```

执行更新：

```bash
apt upgrade
```

完成后，习惯性重启。

### 参考资料

- [PVE 8.0 (Proxmox) 虚拟机系统](https://www.iplaysoft.com/pve.html)
