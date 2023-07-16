---
title: "合并local-lvm和local分区"
linkTitle: "合并local-lvm"
weight: 20
date: 2021-01-18
description: >
  合并local-lvm和local分区
---


## 删除 local-lvm

```bash
lvremove pve/data
```

输出：

```bash
lvremove pve/data
Do you really want to remove active logical volume pve/data? [y/n]: y
  Logical volume "data" successfully removed.
```

## 扩展local


```bash
lvextend -l +100%FREE -r pve/root
```

输出为：

```bash
  Size of logical volume pve/root changed from 96.00 GiB (24576 extents) to 885.25 GiB (226624 extents).
  Logical volume pve/root successfully resized.
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/mapper/pve-root is mounted on /; on-line resizing required
old_desc_blocks = 12, new_desc_blocks = 111
The filesystem on /dev/mapper/pve-root is now 232062976 (4k) blocks long.
```

完成后，查看修改后的状态：

```bash 
df -Th  
```

`/dev/mapper/pve-root` 现在是 831G 了。


```bash
Filesystem           Type      Size  Used Avail Use% Mounted on
udev                 devtmpfs   63G     0   63G   0% /dev
tmpfs                tmpfs      13G  1.9M   13G   1% /run
/dev/mapper/pve-root ext4      871G  4.0G  831G   1% /
tmpfs                tmpfs      63G   66M   63G   1% /dev/shm
tmpfs                tmpfs     5.0M     0  5.0M   0% /run/lock
/dev/nvme0n1p2       vfat     1022M  344K 1022M   1% /boot/efi
/dev/fuse            fuse      128M   48K  128M   1% /etc/pve
tmpfs                tmpfs      13G     0   13G   0% /run/user/0
```

对比修改前的：

```bash
df -Th
Filesystem           Type      Size  Used Avail Use% Mounted on
udev                 devtmpfs   63G     0   63G   0% /dev
tmpfs                tmpfs      13G  2.0M   13G   1% /run
/dev/mapper/pve-root ext4       94G  4.0G   86G   5% /
tmpfs                tmpfs      63G   66M   63G   1% /dev/shm
tmpfs                tmpfs     5.0M     0  5.0M   0% /run/lock
/dev/nvme0n1p2       vfat     1022M  344K 1022M   1% /boot/efi
/dev/fuse            fuse      128M   48K  128M   1% /etc/pve
tmpfs                tmpfs      13G     0   13G   0% /run/user/0

```




## 页面删除Local-LVM

管理页面上，找到 "Datacenter"  -> "Storage"  ，找到 local-lvm ，然后点击删除。

## 修改 local 属性

 "Datacenter"  -> "Storage" 下，选择 local, 编辑 Content ，选中所有内容。
 
 ## 参考资料
 
 - https://alay.cc/701.html

