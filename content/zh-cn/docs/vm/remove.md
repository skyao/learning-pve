---
title: "删除虚拟机"
linkTitle: "删除"
weight: 40
date: 2021-01-18
description: >
  删除虚拟机
---



## 常规方法

标准的做法是在web界面上，将虚拟机停止之后再删除。

## 强制删除

因为某些原因，导致默认的 local-lvm 被删除。此时提交命令

```bash
cd /etc/pve/qemu-server/
rm 100.conf
```

重启，这个虚拟机就消失了。



## 删除虚拟机磁盘文件

某些情况下，比如没有备份的情况下执行 timeshift restore 操作，会导致虚拟机配置文件丢失，而留下 qcow2 格式的虚拟机磁盘文件，如：

```bash
$ pwd
/var/lib/vz/images/104

$ ls -l 
total 23781688
-r--r--r-- 1 root root       917504 Aug 15  2023 base-104-disk-0.qcow2
-r--r--r-- 1 root root 549839962112 Aug 15  2023 base-104-disk-1.qcow2

$ rm -rf base-104-disk-0.qcow2 
rm: cannot remove 'base-104-disk-0.qcow2': Operation not permitted

$ 104 chmod +w base-104-disk-0.qcow2       
chmod: changing permissions of 'base-104-disk-0.qcow2': Operation not permitted$$
```

此时需要用 chattr 去掉 immutable flag 然后再删除：

```bash
chattr -i base-104-disk-0.qcow2
rm base-104-disk-0.qcow2
```

