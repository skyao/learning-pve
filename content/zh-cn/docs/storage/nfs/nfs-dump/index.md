---
title: "[废弃]通过 NFS 进行 dump 和恢复"
linkTitle: "NFS dump"
weight: 30
date: 2021-01-18
description: >
  通过 NFS 进行 dump 和恢复虚拟机和模板内容
---



## 背景

前面纯 nfs 的方案还是手工操作太多。

考虑采用 pve 的 nfs storage，但是只存放 dump 出来的虚拟机或者模板文件，不涉及到 vm id，不会造成冲突。

## 准备工作

### 准备 nfs 服务器端

在 skydev 机器上开通 nfs，额外增加一个 export 地址，为 `/mnt/data/pve-dump`。

## 新建 nfs 存储

打开 pve 管理页面，在 "datacenter" -》 "storage" 下，"Add" 一个名为 "nfs-fast-dump" 的 NFS 存储， export 选择  `/mnt/data/pve-dump`。"content" 只选择 "VZDump backup file"。

此时 pve 会自动在 `/mnt/data/pve-dump` 目录下新建一个名为 "dump" 的子目录，将之前备份的 vma 等文件，复制到  `/mnt/data/pve-dump/dump/` 目录。

刷新 "nfs-fast-dump" 存储的 "Backups" 就可以看到之前备份的 dump 文件，然后就可以用 pve 标准的恢复功能将他们恢复为虚拟机或者模板。

实践了一下, 8G 的模板恢复时间为 66 秒：

```bash
restore vma archive: vma extract -v -r /var/tmp/vzdumptmp101735.fifo /mnt/pve/nfs-fast-dump/dump/vzdump-qemu-107-2023_07_26-02_23_55.vma /var/tmp/vzdumptmp101735
CFG: size: 667 name: qemu-server.conf
DEV: dev_id=1 size: 540672 devname: drive-efidisk0
DEV: dev_id=2 size: 549755813888 devname: drive-scsi0
CTIME: Wed Jul 26 02:23:55 2023
Formatting '/var/lib/vz/images/101/vm-101-disk-0.qcow2', fmt=qcow2 cluster_size=65536 extended_l2=off preallocation=metadata compression_type=zlib size=540672 lazy_refcounts=off refcount_bits=16
new volume ID is 'local:101/vm-101-disk-0.qcow2'
Formatting '/var/lib/vz/images/101/vm-101-disk-1.qcow2', fmt=qcow2 cluster_size=65536 extended_l2=off preallocation=metadata compression_type=zlib size=549755813888 lazy_refcounts=off refcount_bits=16
new volume ID is 'local:101/vm-101-disk-1.qcow2'
map 'drive-efidisk0' to '/var/lib/vz/images/101/vm-101-disk-0.qcow2' (write zeros = 0)
map 'drive-scsi0' to '/var/lib/vz/images/101/vm-101-disk-1.qcow2' (write zeros = 0)
progress 1% (read 5497618432 bytes, duration 1 sec)
progress 2% (read 10995171328 bytes, duration 3 sec)
progress 3% (read 16492724224 bytes, duration 6 sec)
progress 4% (read 21990277120 bytes, duration 7 sec)
......
progress 99% (read 544258785280 bytes, duration 66 sec)
progress 100% (read 549756338176 bytes, duration 66 sec)
total bytes read 549756403712, sparse bytes 541771325440 (98.5%)
space reduction due to 4K zero blocks 5.08%
rescan volumes...
Convert to template.
TASK OK
```

20G 的模板恢复时间为 115 秒：

```bash
restore vma archive: vma extract -v -r /var/tmp/vzdumptmp103025.fifo /mnt/pve/nfs-fast-dump/dump/vzdump-qemu-102-2023_07_26-01_23_50.vma /var/tmp/vzdumptmp103025
CFG: size: 637 name: qemu-server.conf
DEV: dev_id=1 size: 540672 devname: drive-efidisk0
DEV: dev_id=2 size: 549755813888 devname: drive-scsi0
CTIME: Wed Jul 26 01:23:50 2023
Formatting '/var/lib/vz/images/102/vm-102-disk-0.qcow2', fmt=qcow2 cluster_size=65536 extended_l2=off preallocation=metadata compression_type=zlib size=540672 lazy_refcounts=off refcount_bits=16
new volume ID is 'local:102/vm-102-disk-0.qcow2'
Formatting '/var/lib/vz/images/102/vm-102-disk-1.qcow2', fmt=qcow2 cluster_size=65536 extended_l2=off preallocation=metadata compression_type=zlib size=549755813888 lazy_refcounts=off refcount_bits=16
new volume ID is 'local:102/vm-102-disk-1.qcow2'
map 'drive-efidisk0' to '/var/lib/vz/images/102/vm-102-disk-0.qcow2' (write zeros = 0)
map 'drive-scsi0' to '/var/lib/vz/images/102/vm-102-disk-1.qcow2' (write zeros = 0)
progress 1% (read 5497618432 bytes, duration 5 sec)
progress 2% (read 10995171328 bytes, duration 6 sec)
......
progress 98% (read 538761232384 bytes, duration 108 sec)
progress 99% (read 544258785280 bytes, duration 109 sec)
progress 100% (read 549756338176 bytes, duration 115 sec)
total bytes read 549756403712, sparse bytes 529814392832 (96.4%)
space reduction due to 4K zero blocks 2.65%
rescan volumes...
Convert to template.
TASK OK
```

速度只能说还凑合，模板恢复的速度太慢了。

 ## 总结

这个方案中，使用到了 pve nfs 存储的功能，但是为了避免在多个 pve 机器（不是一个 cluster） 之间出现冲突，因此只在这个 nfs 存储中存放不涉及到集群的内容。

## 思路扩展

按照上面的思路，只要 nfs 存储上放的内容不涉及到集群，即不会造成集群范围内的冲突，就不影响使用。

### 扩展存储类型以包含 ISO 文件

因此，nfs 存储存放的类型可以不限于单纯的 "VZDump backup file"， 可以增加 iso image, 毕竟 iso 文件是没有任何集群属性的，而且也没有必要每台 pve 机器都存放一份。

### 手工分配 id 以避免冲突

继续扩展上面的思路，对于可能会造成集群冲突的内容，如果能通过某种方式避免冲突，那么也是可以存放在 nfs 上的（备注：多台 pve 机器但是不建立集群) ，比虚拟机和模板。

关键： 在虚拟机和模板的建立过程（包括新建，clone, 恢复等）中， id 是容许手工指定的。

因此，只要制订并遵守某个 id 分配的规则，让不同 pve 节点上的 nfs 存储以不同的 id 段分配 id, 就可以确保任意一台 pve 节点上的 id 都不会和其他 pve 节点重复。

按照这个思路，修改 nfs 存储可以存放的类型，增加 "Disk image" 用来存放虚拟机和模板的镜像文件。

比如 skysever4 这台机器的 id 就以 14000 开头，从 14001 到 14999，绝对够用。skysever5 就用 15001 到 15999......



