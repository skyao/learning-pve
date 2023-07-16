---
title: "pmxcfs介绍"
linkTitle: "介绍"
weight: 100
date: 2021-01-18
description: >
  pmxcfs是一个数据库驱动的文件系统，用于存储配置文件
---


参考：

https://pve.proxmox.com/pve-docs/pve-admin-guide.html#chapter_pmxcfs

## pmxcfs

Proxmox 集群文件系统 （“pmxcfs”） 是一个数据库驱动的文件系统，用于存储配置文件，使用 corosync 实时复制到所有集群节点。我们使用它来存储所有与Proxmox VE相关的配置文件。

尽管文件系统将所有数据存储在磁盘上的持久数据库中，但数据的副本驻留在 RAM 中。这对最大大小施加了限制，目前为 128 MiB。这仍然足以存储数千个虚拟机的配置。

该系统具有以下优点：

- 将所有配置实时无缝复制到所有节点
- 提供强大的一致性检查以避免重复的虚拟机 ID
- 节点丢失仲裁时为只读
- 自动更新所有节点的同步群集配置
- 包括分布式锁定机制



文件系统挂载在：

```
/etc/pve
```



