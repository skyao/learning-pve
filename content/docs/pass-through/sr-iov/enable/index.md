---
title: "配置网卡以开启 SR-IOV"
linkTitle: "开启 SR-IOV"
weight: 20
date: 2021-01-18
description: >
  配置网卡配置以便在 pve 中开启网卡的 SR-IOV 功能
---



## 准备工作

### 更新网卡固件

对于 HP544+ 网卡，参考：

https://skyao.io/learning-computer-hardware/nic/hp544/firmware/

### 设置主板bios

需要在主板中设置开启以下功能：

- 虚拟机支持
- vt-d
- SR-IOV

### 安装 pve-headers

对于 PVE，还需要安装额外的 pve-headers，否则后续安装 mft 时会报错，提示要安装 linux-headers 和 linux-headers-generic：

```bash
./install.sh
-E- There are missing packages that are required for installation of MFT.
-I- You can install missing packages using: apt-get install linux-headers linux-headers-generic
```

这两个包还不能直接用 `apt install` 进行安装：

```bash
$ apt install linux-headers  

Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package linux-headers is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source

E: Package 'linux-headers' has no installation candidate
```

正确的安装方式是安装 pve-headers，但默认情况下 pve-headers 也是找不到的：

```bash
$ apt apt install pve-headers  

Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package pve-headers
```

需要添加 pve-no-subscriptio 仓库才行。

```bash
vi /etc/apt/sources.list.d/pve-no-subscription.list
```

设置内容：

```properties
deb https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian bookworm pve-no-subscription
```

`apt update` 之后就可以安装 pve-headers 了：

```bash
apt install pve-headers                            
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  pve-headers-6.2 pve-headers-6.2.16-4-pve pve-kernel-6.2 pve-kernel-6.2.16-4-pve
The following NEW packages will be installed:
  pve-headers pve-headers-6.2 pve-headers-6.2.16-4-pve pve-kernel-6.2.16-4-pve
The following packages will be upgraded:
  pve-kernel-6.2
1 upgraded, 4 newly installed, 0 to remove and 26 not upgraded.
Need to get 111 MB of archives.
After this operation, 659 MB of additional disk space will be used.
Do you want to continue? [Y/n] 
```

安装完成之后，需要重启，否则直接安装 mft，依然会继续同样报错：

```bash
./install.sh 
-E- There are missing packages that are required for installation of MFT.
-I- You can install missing packages using: apt-get install linux-headers linux-headers-generic
```

这里描述了 linux-headers 和 pve-headers-6.2 的关系：  linux-headers 只是一个虚拟的包，有多个提供者，在当前6.2版本的 pve 上就是 pve-headers-6.2 ：

```bash
apt-get install linux-headers linux-headers-generic
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package linux-headers-generic is a virtual package provided by:
  pve-headers-6.2 8.0.3
  pve-headers-6.1 7.3-4
  linux-headers-amd64 6.1.38-1
You should explicitly select one to install.

Package linux-headers is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source

E: Package 'linux-headers' has no installation candidate
E: Package 'linux-headers-generic' has no installation candidate
```

### 安装 mft 

需要 mft 工具修改 ConnectX 网卡的配置，下载安装方式为：

```bash
wget --no-check-certificate https://www.mellanox.com/downloads/MFT/mft-4.24.0-72-x86_64-deb.tgz
tar xvf mft-4.24.0-72-x86_64-deb.tgz
cd mft-4.24.0-72-x86_64-deb
```

如果缺少依赖包会导致安装失败，可以先apt命令安装以下包 （以及前面的 pve-headers）：

```bash
apt-get install gcc make dkms
./install.sh
```

成功时的输出为：

```bash
-I- Removing mft external packages installed on the machine
-I- Installing package: /root/temp/mft-4.24.0-72-x86_64-deb/SDEBS/kernel-mft-dkms_4.24.0-72_all.deb
-I- Installing package: /root/temp/mft-4.24.0-72-x86_64-deb/DEBS/mft_4.24.0-72_amd64.deb
-I- In order to start mst, please run "mst start".
```

### 下载 mlxup

```bash
wget https://www.mellanox.com/downloads/firmware/mlxup/4.22.1/SFX/linux_x64/mlxup
chmod +x ./mlxup
./mlxup
```

能看到当前网卡的固件情况：

```bash
Querying Mellanox devices firmware ...

Device #1:
----------

  Device Type:      ConnectX3Pro
  Part Number:      764285-B21_Ax
  Description:      HP InfiniBand FDR/Ethernet 10Gb/40Gb 2-port 544+FLR-QSFP Adapter
  PSID:             HP_1380110017
  PCI Device Name:  /dev/mst/mt4103_pci_cr0
  Port1 GUID:       c4346bffffdfe181
  Port2 GUID:       c4346bffffdfe182
  Versions:         Current        Available     
     FW             2.42.5700      N/A           

  Status:           No matching image found
```

### 安装 mstflint

```bash
apt install mstflint
```





## 设置网卡

### 启动 mft 工具

```bash
mst start
```

输入为：

```bash
Starting MST (Mellanox Software Tools) driver set
Loading MST PCI module - Success
Loading MST PCI configuration module - Success
Create devices
```

### 查看网卡配置

```bash
mst status
```

可以看到当前网卡信息（这是只插了一块HP544+ 网卡的情况):

```bash
MST modules:
------------
    MST PCI module loaded
    MST PCI configuration module loaded

MST devices:
------------
/dev/mst/mt4103_pciconf0         - PCI configuration cycles access.
                                   domain:bus:dev.fn=0000:02:00.0 addr.reg=88 data.reg=92 cr_bar.gw_offset=-1
                                   Chip revision is: 00
/dev/mst/mt4103_pci_cr0          - PCI direct access.
                                   domain:bus:dev.fn=0000:02:00.0 bar=0xdfe00000 size=0x100000
                                   Chip revision is: 00
```

继续执行命令：

```bash
mlxconfig -d /dev/mst/mt4103_pciconf0 q
```

可以看到网卡的配置信息：

```bash
Device #1:
----------

Device type:    ConnectX3Pro    
Device:         /dev/mst/mt4103_pciconf0

Configurations:                                      Next Boot
         SRIOV_EN                                    True(1)         
         NUM_OF_VFS                                  16              
         WOL_MAGIC_EN_P2                             True(1)         
         LINK_TYPE_P1                                VPI(3)          
         PHY_TYPE_P1                                 0               
         XFI_MODE_P1                                 _10G(0)         
         FORCE_MODE_P1                               False(0)        
         LINK_TYPE_P2                                VPI(3)          
         PHY_TYPE_P2                                 0               
         XFI_MODE_P2                                 _10G(0)     
```

### 修改网卡配置

```bash
mlxconfig -d /dev/mst/mt4103_pciconf0 set SRIOV_EN=1 NUM_OF_VFS=8
```

对于有多块网卡的机器，可以根据需要决定是否为其中的一块或者多块网卡开启 SR-IOV ，然后需要开启的设置 SRIOV_EN=1 ，不需要开启的设置 SRIOV_EN=0。

注意这里的 mt4103_pciconf0 / mt4103_pciconf1 等编号是按照 lspci 时显示的设备 id 来排序的，因此用这个方法可以分辨各个网卡。

重启机器。

## 设置网卡驱动

```bash
vi /etc/modprobe.d/mlx4_core.conf
```

输入内容：

```bash
options mlx4_core num_vfs=4,4,0 port_type_array=2,2 probe_vf=4,4,0 probe_vf=4,4,0
options mlx4_core enable_sys_tune=1
options mlx4_en inline_thold=0
options mlx4_core log_num_mgm_entry_size=-7
```

执行：

```bash
update-initramfs -u
```

## 开启 SR-IOV

开启成功时的例子，这里有两块网卡，一块设置开启 SR-IOV，另一块设置不开启 SR-IOV （准备网卡整体直通给虚拟机）。

```bash
lspci | grep Mel
01:00.0 Ethernet controller: Mellanox Technologies MT27520 Family [ConnectX-3 Pro]
04:00.0 Ethernet controller: Mellanox Technologies MT27520 Family [ConnectX-3 Pro]
04:00.1 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
04:00.2 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
04:00.3 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
04:00.4 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
04:00.5 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
04:00.6 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
04:00.7 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
04:01.0 Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]
```

-  `01:00.0 Ethernet controller: Mellanox Technologies MT27520 Family [ConnectX-3 Pro]` ： 设置为不开启 SR-IOV 的网卡，显示为一块物理网卡（PF）。
- `04:00.0 Ethernet controller: Mellanox Technologies MT27520 Family [ConnectX-3 Pro]`： 设置为开启 SR-IOV 的网卡，显示为一块物理网卡（PF） + 下面多块（例子中是每个port 4个，一共8个） 虚拟网卡（VF）。
- `04:00.x Ethernet controller: Mellanox Technologies MT27500/MT27520 Family [ConnectX-3/ConnectX-3 Pro Virtual Function]`： 设置为开启 SR-IOV 的网卡的所有的 VF 。

## 参考资料

- [How-to: configure Mellanox ConnectX-3 cards for SRIOV and VFS](https://forum.proxmox.com/threads/how-to-configure-mellanox-connectx-3-cards-for-sriov-and-vfs.121927/): 这个帖子对此有非常详细的讲解，我基本是按照他的指点来操作的

