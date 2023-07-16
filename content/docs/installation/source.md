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

### ~~给 PVE 更换国内源~~

备注：这个先不加了，会导致更新过于超前。比如这里会推送 pve 8.0.3 ，更新后 pvetools 工具不支持。pve 稳定就好，不追新。 

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

顺便安装一些常用的软件：

```bash
apt install unzip net-tools
```

完成后，习惯性重启。

### 降级pve版本

查看当前 pve 版本：

```bash
$ pveversion
pve-manager/8.0.3/bbf3993334bfa916 (running kernel: 6.2.16-4-pve)
```

列举可选的 pve 版本：

```bash
apt policy pve-manager
```

没有 8.0.2：

```bash
pve-manager:
  Installed: 8.0.3
  Candidate: 8.0.3
  Version table:
 *** 8.0.3 500
        500 https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian bookworm/pve-no-subscription amd64 Packages
        100 /var/lib/dpkg/status
     8.0.0~9 500
        500 https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian bookworm/pve-no-subscription amd64 Packages
     8.0.0~8 500
        500 https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian bookworm/pve-no-subscription amd64 Packages

```

只好想降级到 8.0.0~9 ： 

```bash
apt install pve-manager=8.0.0~9
```

输出如下：

```bash
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Suggested packages:
  libpve-network-perl
Recommended packages:
  proxmox-offline-mirror-helper
The following packages will be DOWNGRADED:
  pve-manager
0 upgraded, 0 newly installed, 1 downgraded, 0 to remove and 0 not upgraded.
Need to get 499 kB of archives.
After this operation, 13.3 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian bookworm/pve-no-subscription amd64 pve-manager amd64 8.0.0~9 [499 kB]
Fetched 499 kB in 0s (1,190 kB/s)   
dpkg: warning: downgrading pve-manager from 8.0.3 to 8.0.0~9
(Reading database ... 55730 files and directories currently installed.)
Preparing to unpack .../pve-manager_8.0.0~9_amd64.deb ...
Unpacking pve-manager (8.0.0~9) over (8.0.3) ...
Setting up pve-manager (8.0.0~9) ...
Adding pvetest repo to '/etc/apt/sources.list.d/pvetest-for-beta.list' to enable updates during Proxmox VE 8.0 BETA
deb http://download.proxmox.com/debian/pve bookworm pvetest
Processing triggers for man-db (2.11.2-2) ...
```





### 参考资料

- [PVE 8.0 (Proxmox) 虚拟机系统](https://www.iplaysoft.com/pve.html)
