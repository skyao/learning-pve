---
title: "[废弃]通过NFS复制"
linkTitle: "NFS复制"
weight: 20
date: 2021-01-18
description: >
  通过 NFS 复制文件来同步虚拟机和模板内容
---



## 背景

scp 命令的速度，只能达到 300-400MB,无法发挥出网络和硬盘的速度。

nfs 会快一些，实际测试可以达到 2000 - 3000 MB （主要瓶颈在ssd写入速度）。

## 准备工作

### 准备 nfs 服务器端

在 skydev 机器上开通 nfs，然后将 4t 的 pcie 4.0 ssd 以 nfs 的方式共享出来

nfs 地址为： 192.168.99.100 ， export 地址为 `/mnt/data/share`。

### 准备 nfs 客户端

pve 节点上安装 nfs 客户端。

### mount nfs 到 pve 节点

在pve 节点上执行命令：

```bash
mkdir -p /mnt/nfs-fast
# on skywork
sudo mount 192.168.100.1:/mnt/data/share /mnt/nfs-fast
# on skyserver
mount 192.168.99.100:/mnt/data/share /mnt/nfs-fast
```

为了方便，

```bash
vi ~/.zshrc
```

增加一个 alias :

```bash
# on skywork
alias mount-nfs-fast='sudo mount 192.168.100.1:/mnt/data/share /mnt/nfs-fast'
# on skyserver
alias mount-nfs-fast='mount 192.168.99.100:/mnt/data/share /mnt/nfs-fast'
```

## 复制虚拟机和模板

复制虚拟机或者模板文件到 pve 的 dump 目录: 

```bash
cd /mnt/nfs-fast/pve-share/templates/ubuntu20.04
cp *.vma *.vma.notes /var/lib/vz/dump 
```

同样为了方便，

```bash
vi ~/.zshrc
```

增加一个 alias :

```bash
alias copy-pve-dump='cp *.vma *.vma.notes /var/lib/vz/dump/'
```

### 恢复虚拟机或者模板

在 pve 管理页面操作，打开 local，"Backups" 。

 ## 总结

这个方案中，nfs 只是作为一种快速的远程源文件复制方式，从 nfs 服务器端将虚拟机或者模板文件复制到 pve 本地存储，然后通过标准的 pve 虚拟机恢复方式来恢复虚拟机或者模板。之后虚拟机或者模板就存放在 pve 本地存储上了，再进行各种 clone 操作也就方便了。

缺点就是完全绕开了 pve, 需要 ssh + 命令手工执行。



