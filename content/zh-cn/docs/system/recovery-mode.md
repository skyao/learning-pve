---
title: "进入恢复模式修复系统"
linkTitle: "恢复模式"
weight: 70
date: 2021-01-18
description: >
  在必要时通过进入 pve 的恢复模式来修复系统
---

## 背景

有时在经过错误的配置之后，pve 系统会无法使用，甚至无法进入系统和页面控制台。

此时需要有其他的方案来修复系统。

## 恢复模式

### 进入恢复模式

开机，选择 "Advanced options for Proxmox VE GNU/Linux" ，然后选择 Rescue mode。

用 root 密码登录即可。

### 启动集群

```bash
systemctl start pve-cluster
```

这是就可以访问目录 `/etc/pve` 来操作集群和虚拟机了。

### 操作虚拟机

比如关闭有问题的虚拟机：

```bash
qm stop id
```

### 修改虚拟机配置

```bash
cd /etc/pve/nodes/<node_name>/qume-server
```

这里能看到各个虚拟机的配置文件，打开相应的配置文件修改即可，比如取消虚拟机的开机，只要将 onboot 修改为 0.
