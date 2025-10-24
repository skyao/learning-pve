---
title: "配置国内的更新源"
linkTitle: "更新源"
weight: 30
date: 2023-07-09
description: >
  配置国内的更新源
---

## pve8

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

## pve9

PVE 9.0 基于 Debian 13，除了换 Debian 的软件源以外，还需要编辑企业源、Ceph 源、无订阅源以及 CT 模板源。

### 设置 debian 源

修改之前先备份一下：

```bash
cp -r /etc/apt/sources.list.d/ /etc/apt/sources.list.d.original
```

修改 debian 源：

```bash
vi /etc/apt/sources.list.d/debian.sources
```

删除原有内容，添加如下内容：

```properties
Types: deb
URIs: https://mirrors.tuna.tsinghua.edu.cn/debian
Suites: trixie trixie-updates trixie-backports
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
# Types: deb-src
# URIs: https://mirrors.tuna.tsinghua.edu.cn/debian
# Suites: trixie trixie-updates trixie-backports
# Components: main contrib non-free non-free-firmware
# Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
Types: deb
URIs: https://security.debian.org/debian-security
Suites: trixie-security
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

# Types: deb-src
# URIs: https://security.debian.org/debian-security
# Suites: trixie-security
# Components: main contrib non-free non-free-firmware
# Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
```

### 删除企业源

```bash
rm -rf /etc/apt/sources.list.d/pve-enterprise.sources
```

### Ceph 源

修改 Ceph 源：

```bash
vi /etc/apt/sources.list.d/ceph.sources
```

删除原有内容，添加如下内容：

```properties
Types: deb
URIs: https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian/ceph-squid
Suites: trixie
Components: no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```

### 无订阅源

```bash
vi /etc/apt/sources.list.d/pve-no-subscription.sources
```

内容如下：

```properties
Types: deb
URIs: https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian/pve
Suites: trixie
Components: pve-no-subscription
Signed-By: /usr/share/keyrings/proxmox-archive-keyring.gpg
```

### CT 模板源

如果需要用到 PVE 中的 LXC 容器，则需要替换 CT 模板源：

```bash
cp /usr/share/perl5/PVE/APLInfo.pm /usr/share/perl5/PVE/APLInfo.pm.original
sed -i 's|http://download.proxmox.com|https://mirrors.tuna.tsinghua.edu.cn/proxmox|g' /usr/share/perl5/PVE/APLInfo.pm
```

### 更新

```bash
apt update
apt upgrade
```

