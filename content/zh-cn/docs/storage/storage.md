---
title: "存储介绍"
linkTitle: "介绍"
weight: 10
date: 2021-01-18
description: >
  介绍PVE的存储
---

参考：

https://pve.proxmox.com/pve-docs/pve-admin-guide.html#chapter_storage

Proxmox VE存储模型非常灵活。虚拟机映像可以存储在一个或多个本地存储上，也可以存储在 NFS 或 iSCSI（NAS、SAN）等共享存储上。没有限制，您可以根据需要配置任意数量的存储池。您可以使用所有可用于 Debian Linux 的存储技术。

将 VM 存储在共享存储上的一个主要好处是能够在不停机的情况下实时迁移正在运行的计算机，因为群集中的所有节点都可以直接访问 VM 磁盘映像。无需复制 VM 映像数据，因此在这种情况下，实时迁移非常快。

存储库（包libpve-storage-perl）使用灵活的插件系统为所有存储类型提供通用接口。这可以很容易地采用，以包括将来的更多存储类型。



## 存储类型

基本上有两种不同类型的存储类型：

- File level storage 文件级存储

  基于文件级的存储技术允许访问功能齐全的 （POSIX） 文件系统。它们通常比任何块级存储（见下文）更灵活，并允许您存储任何类型的内容。ZFS 可能是最先进的系统，它完全支持快照和克隆。

- Block level storage 块级存储

  允许存储大型 raw 镜像。通常不可能在此类存储类型上存储其他文件（ISO、备份等）。大多数现代块级存储实现都支持快照和克隆。RADOS和GlusterFS是分布式系统，将存储数据复制到不同的节点。

### 可用的存储类型

| Description    | Plugin type | Level | Shared | Snapshots | Stable             |
| :------------- | :---------- | :---- | :----- | :-------- | :----------------- |
| ZFS (local)    | zfspool     | both1 | no     | yes       | yes                |
| Directory      | dir         | file  | no     | no2       | yes                |
| BTRFS          | btrfs       | file  | no     | yes       | technology preview |
| NFS            | nfs         | file  | yes    | no2       | yes                |
| CIFS           | cifs        | file  | yes    | no2       | yes                |
| Proxmox Backup | pbs         | both  | yes    | n/a       | yes                |
| GlusterFS      | glusterfs   | file  | yes    | no2       | yes                |
| CephFS         | cephfs      | file  | yes    | yes       | yes                |
| LVM            | lvm         | block | no3    | no        | yes                |
| LVM-thin       | lvmthin     | block | no     | yes       | yes                |
| iSCSI/kernel   | iscsi       | block | yes    | no        | yes                |
| iSCSI/libiscsi | iscsidirect | block | yes    | no        | yes                |
| Ceph/RBD       | rbd         | block | yes    | yes       | yes                |
| ZFS over iSCSI | zfs         | block | yes    | yes       | yes                |



### 精简配置

许多存储和 QEMU 映像格式 qcow2 支持精简配置。激活精简置备后，只有客户机系统实际使用的块才会写入存储。

例如，假设您创建了一个具有 32GB 硬盘的 VM，在安装 guest 系统操作系统后，该 VM 的根文件系统包含 3 GB 的数据。在这种情况下，即使来宾虚拟机看到 32GB 的硬盘驱动器，也只会将 3GB 写入存储。通过这种方式，精简配置允许您创建大于当前可用存储块的磁盘映像。可以为 VM 创建大型磁盘映像，并在需要时向存储添加更多磁盘，而无需调整 VM 文件系统的大小。

具有“快照”功能的所有存储类型也支持精简配置。



## 存储配置

所有与Proxmox VE相关的存储配置都存储在/etc/pve/storage.cfg的单个文本文件中。由于此文件位于 /etc/pve/ 中，因此会自动分发到所有群集节点。因此，所有节点共享相同的存储配置。

共享存储配置对于共享存储非常有意义，因为可以从所有节点访问相同的“共享”存储。但它对于本地存储类型也很有用。在这种情况下，这种本地存储在所有节点上都可用，但它在物理上是不同的，并且可以具有完全不同的内容。

### 存储池

每个存储池都有一个  `<type>`，并由其 `<STORAGE_ID>`。池配置如下所示：

```properties
<type>: <STORAGE_ID>
        <property> <value>
        <property> <value>
        <property>
        ...
```

`<type>: <STORAGE_ID>` 行开始池定义，然后是属性列表。大多数属性都需要值。有些具有合理的默认值，在这种情况下，您可以省略该值。

更具体地说，请查看安装后的默认存储配置。它包含一个名为 local 的特殊本地存储池，该池引用目录 /var/lib/vz 并且始终可用。Proxmox VE安装程序根据安装时选择的存储类型创建其他存储条目。

#### 默认存储配置 （/etc/pve/storage.cfg）

```properties
dir: local
        path /var/lib/vz
        content iso,vztmpl,backup

# default image store on LVM based installation
lvmthin: local-lvm
        thinpool data
        vgname pve
        content rootdir,images

# default image store on ZFS based installation
zfspool: local-zfs
        pool rpool/data
        sparse
        content images,rootdir
```

