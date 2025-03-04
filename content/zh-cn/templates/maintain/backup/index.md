---
title: "模板备份"
linkTitle: "备份"
weight: 50
date: 2025-03-04
description: >
  debian pve 模板的备份
---

### 虚拟机备份

参考: https://skyao.io/learning-pve/docs/vm/backup/backup/

pve下虚拟机备份的目录在: `/var/lib/vz/dump/`

```bash
$ ls -lh /var/lib/vz/dump/
total 2.7G
-rw-r--r-- 1 root root 5.1K Mar  4 11:28 vzdump-qemu-10102-2025_03_04-11_26_15.log
-rw-r--r-- 1 root root 2.7G Mar  4 11:28 vzdump-qemu-10102-2025_03_04-11_26_15.vma.zst
-rw-r--r-- 1 root root   91 Mar  4 11:28 vzdump-qemu-10102-2025_03_04-11_26_15.vma.zst.notes
```

### 备份文件的存储

虚拟机的备份文件分别保存在本地，移动硬盘和NAS中。

最方便的是用 scp 命令复制备份文件：

```bash
scp ./vzdump-qemu-102-2023_07_26-01_23_50.* sky@192.168.0.240:/media/sky/data/backup/pve

scp root@192.168.20.29:"/var/lib/vz/dump/vzdump-*" /Volumes/u4t/data/backup/pve-backup/skyaio2
```









