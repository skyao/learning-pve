---
title: "安装配置timeshift"
linkTitle: "timeshift"
weight: 40
date: 2021-01-18
description: >
  安装配置 timeshift 来对 PVE 进行备份和恢复
---

前面在安装 pve 时，选择了留出 50G 大小的空间给 timeshift 使用。

## 为 timeshift 准备分区

### 查看当前磁盘分区情况

```bash
fdisk -l
```

查看当前硬盘情况：

```bash
Disk /dev/nvme0n1: 465.76 GiB, 500107862016 bytes, 976773168 sectors
Disk model: WDC WDS500G1B0C-00S6U0                  
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 000372CE-8B4D-4B12-974B-99C231DB3D67

Device           Start       End   Sectors  Size Type
/dev/nvme0n1p1      34      2047      2014 1007K BIOS boot
/dev/nvme0n1p2    2048   2099199   2097152    1G EFI System
/dev/nvme0n1p3 2099200 838860800 836761601  399G Linux LVM


Disk /dev/mapper/pve-swap: 8 GiB, 8589934592 bytes, 16777216 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/pve-root: 96 GiB, 103079215104 bytes, 201326592 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

这是一个大小为 480G 的 nvme SSD 硬盘，实际可用空间为 465G。在安装 PVE 时我将 hdsize 设置为 400G, 因此 PVE 使用的空间只有400G。从上面输出可以看到：

```bash
/dev/nvme0n1p1      34      2047      2014 1007K BIOS boot
/dev/nvme0n1p2    2048   2099199   2097152    1G EFI System
/dev/nvme0n1p3 2099200 838860800 836761601  399G Linux LVM
```

nvme0n1p1 是 BIOS， nvme0n1p2 是 EFI 分区，PVE 使用的是 nvme0n1p3，399G。

另外可以看到：

```bash
Disk /dev/mapper/pve-swap: 8 GiB, 8589934592 bytes, 16777216 sectors
Disk /dev/mapper/pve-root: 96 GiB, 103079215104 bytes, 201326592 sectors
```

swap 使用了 8G， pve root 使用了 96 G。

```bash
lsblk -f
```

可以看到 nvme0n1p3 是一个 LVM2，400G 的总空间中，内部被分为 pve-swap / pve-root / pve-data_tmeta

```bash

NAME               FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
nvme0n1                                                                                             
|-nvme0n1p1                                                                                         
|-nvme0n1p2        vfat        FAT32          26F2-091E                              1021.6M     0% /boot/efi
`-nvme0n1p3        LVM2_member LVM2 001       foXyBZ-IPQT-wOCF-DWSN-u88a-sBVD-LaZ9zn                
  |-pve-swap       swap        1              c5d97cb5-4d18-4f43-a36f-fb67eb8dcc84                  [SWAP]
  |-pve-root       ext4        1.0            a0a46add-dda5-42b4-a978-97d363eeddd0     85.2G     4% /
  |-pve-data_tmeta                                                                                  
  | `-pve-data                                                                                      
  `-pve-data_tdata                                                                                  
    `-pve-data   
```

剩余的大约 60G 的空间在这里没有显示。

### 添加 timeshift 分区

```bash
fdisk /dev/nvme0n1
```

输出为：

```bash
Welcome to fdisk (util-linux 2.38.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

This disk is currently in use - repartitioning is probably a bad idea.
It's recommended to umount all file systems, and swapoff all swap
partitions on this disk.
```

在警告说磁盘使用中，重新分区不是一个好注意。不过我们是操作剩余空间，比较安全。

输入 m 获得帮助：

```bash
Command (m for help): m

Help:

  GPT
   M   enter protective/hybrid MBR

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty MBR (DOS) partition table
   s   create a new empty Sun partition table
```

输入 F 列出未分区的剩余空间：

```bash
Command (m for help): F

Unpartitioned space /dev/nvme0n1: 65.76 GiB, 70610066944 bytes, 137910287 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes

    Start       End   Sectors  Size
838862848 976773134 137910287 65.8G
```

这个 65.8G 就是我们之前预留的空间。

输入 n 添加新的分区，默认是利用剩余空间的全部。这里敲三次回车全部默认即可。

```bash
Command (m for help): n
Partition number (4-128, default 4): 
First sector (838860801-976773134, default 838862848): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (838862848-976773134, default 976773119): 

Created a new partition 4 of type 'Linux filesystem' and of size 65.8 GiB.
```

如果不是利用全部剩余空间，则可以通过输入 `+100G` 这样的内容来设置要添加的分区大小。

输入 p 显示修改之后的分区表：

```bash
Command (m for help): p
Disk /dev/nvme0n1: 465.76 GiB, 500107862016 bytes, 976773168 sectors
Disk model: WDC WDS500G1B0C-00S6U0                  
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 000372CE-8B4D-4B12-974B-99C231DB3D67

Device             Start       End   Sectors  Size Type
/dev/nvme0n1p1        34      2047      2014 1007K BIOS boot
/dev/nvme0n1p2      2048   2099199   2097152    1G EFI System
/dev/nvme0n1p3   2099200 838860800 836761601  399G Linux LVM
/dev/nvme0n1p4 838862848 976773119 137910272 65.8G Linux filesystem
```

可以看到我们新加的 nvme0n1p4 分区大小为 65.8G。

检查无误，输入 w 写入修改操作并退出。

```bash
Command (m for help): w
The partition table has been altered.
Syncing disks.
```

在 pve 下执行

```bash
fdisk -l 
```

再次检查分区情况：

```bash
......
Device             Start       End   Sectors  Size Type
/dev/nvme0n1p1        34      2047      2014 1007K BIOS boot
/dev/nvme0n1p2      2048   2099199   2097152    1G EFI System
/dev/nvme0n1p3   2099200 838860800 836761601  399G Linux LVM
/dev/nvme0n1p4 838862848 976773119 137910272 65.8G Linux filesystem
```

至此 timeshift 分区准备OK.

### 格式化 timeshift 分区

```bash
mkfs.ext4 /dev/nvme0n1p4
```

对分区进行格式化，格式为 ext4：

```bash

mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done                            
Creating filesystem with 17238784 4k blocks and 4317184 inodes
Filesystem UUID: 2411eb4e-67f1-4c7d-b633-17df1fa0c127
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (131072 blocks): done
Writing superblocks and filesystem accounting information: done 
```



## 安装 timeshift

具体参考这里：

https://skyao.io/learning-ubuntu-server/docs/installation/timeshift/install

安装配置完成后，执行

```bash
timeshift --list
```

查看 timeshift 情况：

```bash

First run mode (config file not found)
Selected default snapshot type: RSYNC
Mounted '/dev/nvme0n1p4' at '/run/timeshift/5364/backup'
Device : /dev/nvme0n1p4
UUID   : 2411eb4e-67f1-4c7d-b633-17df1fa0c127
Path   : /run/timeshift/5364/backup
Mode   : RSYNC
Status : No snapshots on this device
First snapshot requires: 0 B

No snapshots found
```



## 使用 timeshift 进行备份

### 设置自动备份

```bash
vi /etc/timeshift/timeshift.json
```

打开 timeshift 配置文件，修改以下内容

```yaml
{
  "schedule_monthly" : "true",
  "schedule_weekly" : "true",
  "schedule_daily" : "true",
  "schedule_hourly" : "false",
  "schedule_boot" : "false",
  "count_monthly" : "2",
  "count_weekly" : "3",
  "count_daily" : "5",
  "count_hourly" : "6",
  "count_boot" : "5",
}

```

对于软路由机器，由于是长期开机并且改动不过，因此 schedule_monthly / schedule_weekly /  schedule_daily 设置为 true，数量用默认值。

### 排除不需要备份的内容

默认情况下 timeshift 的配置文件中已经设置了 exclude：

```bash
{
  "exclude" : [
    "/var/lib/ceph/**",
    "/root/**"
  ],
  "exclude-apps" : []
}

```

- `/var/lib/ceph/`： ceph 相关的文件

- `/root/`: root 用户的 home 路径，如果使用其他用户，也需要将它们的 home 目录加入进来。

但这些是不够的，还需要增加以下内容：

- `/var/lib/vz/`

  pve的各种文件，包括虚拟机，模版，备份，上传的iso文件等都在这里，这些文件太大，不适合用 timeshift 备份，因此必须排除。

  ```bash
  ls /var/lib/vz
  dump  images  private  snippets  template
  ```

  - dump： 这里保存 pve backup 备份下来的虚拟机镜像文件
  - images： 这里保存 pve 虚拟机镜像文件
  - template： iso 子目录中保存的是上传到 pve 的各种 iso/img 文件

- `/etc/pve/qemu-server/`: pve 的虚拟机配置文件，这些文件也不要用 timeshift 备份，避免恢复时把虚拟机文件也给覆盖了。虚拟机文件的备份会由下一节中提到的自动脚本进行备份。

- `/mnt/pve`: 在 pve 下使用 nfs 存储时，远程 nfs 会自动 mount 到这个目录下，这些文件肯定也不能被 timeshift 备份。因此必须排除。 

最后通过设置的 exclude 为：

```bash
{
  ......
  "exclude" : [
    "/var/lib/ceph/**",
    "/root/**",
    "/var/lib/vz/**",
    "/etc/pve/qemu-server/**",
    "/mnt/pve/**"
  ],
}
```

### 设置自动备份虚拟机文件

见下一章，推荐完成这个操作之后再执行手工备份。

### 手工备份

```bash
timeshift --create --comments "first backup after apt upgrade and basic soft install"
```

输入为：

```bash
Estimating system size...
Creating new snapshot...(RSYNC)
Saving to device: /dev/nvme0n1p4, mounted at path: /run/timeshift/6888/backup
Syncing files with rsync...
Created control file: /run/timeshift/6888/backup/timeshift/snapshots/2023-07-25_21-10-04/info.json
RSYNC Snapshot saved successfully (41s)
Tagged snapshot '2023-07-25_21-10-04': ondemand
------------------------------------------------------------------------------
```

查看当前备份情况：

```bash
timeshift --list                                                                      
Mounted '/dev/nvme0n1p4' at '/run/timeshift/7219/backup'
Device : /dev/nvme0n1p4
UUID   : 2411eb4e-67f1-4c7d-b633-17df1fa0c127
Path   : /run/timeshift/7219/backup
Mode   : RSYNC
Status : OK
1 snapshots, 64.5 GB free

Num     Name                 Tags  Description                                            
------------------------------------------------------------------------------
0    >  2023-07-25_21-10-04  O     first backup after apt upgrade and basic soft install  
```

结合备份之前看到的 timeshift 备份分区的大小为 65.8G，减去这里的 64.5 GB free，也就是这个备份用掉了 1.3 G 的磁盘空间。
