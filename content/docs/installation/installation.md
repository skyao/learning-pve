---
title: "安装PVE"
linkTitle: "安装"
weight: 20
date: 2023-07-09
description: >
  在物理机上安装PVE
---



## 刻录安装U盘

用下载下来的 iso 文件，刻录安装U盘。

## 安装

设置 bios 为从 u 盘启动，推荐 UEFI 模式。

全程按照提示操作即可。

https://pve.proxmox.com/pve-docs/pve-admin-guide.html#installation_installer

安装程序将创建一个名为 pve 的卷组 （VG），以及称为根卷、数据和交换的其他逻辑卷 （LV）。要控制这些卷的大小，请使用：

- hdsize / 硬盘大小

  定义要使用的总硬盘大小。这样，您可以在硬盘上保留可用空间以进行进一步分区（例如，在同一硬盘上用于 LVM 存储的额外 PV 和 VG）。

- swapsize / 交换大小

  定义交换卷的大小。默认值为已安装内存的大小，最小 4 GB，最大 8 GB。结果值不能大于 hdsize/8。

- maxroot / 根卷的最大大小

  定义存储操作系统的根卷的最大大小。根卷大小的最大限制为 hdsize/4。

- maxvz / 最大VZ

  定义数据卷的最大大小。数据量的实际大小为：

  datasize = hdsize - rootsize - swapsize - minfree
  其中 datasize 不能大于 maxvz。

  > 注意： 如果设置为 0，则不会创建任何数据卷，并且将相应地调整存储配置。

- minfree 

  定义 LVM 卷组 pve 中剩余的可用空间量。当可用的存储空间超过 128GB 时，默认值为 16GB，否则将使用 hdsize/8。

暂时用的全默认值进行安装，PVE installer 会创建 local 和 local-lvm， 大小大概是 100g 和 剩余空间。swapsize 为 8G.

> 备注：zfs 和 ceph 的使用稍后再尝试。目前没有开启

为了支持使用 timeshift 工具进行备份和恢复，在安装时选择留出部分空间给到 timeshift，因此 hdsize 被设置为实际硬盘大小 - 50G 左右。后面再对这个空余空间进行分区和格式化并安装和配置 timeshift 。

## 常见问题

### "Starting the installer GUI"

安装开始后，在获取 DHCP 之后，会准备启动图形界面的安装器。然后就在这里失败，无法启动图形界面。

参考： 

- [Proxmox 6.2-1 installation fails after DHCP lease obtained](https://www.rpiathome.com/2020/10/21/proxmox-6-2-1-installation-fails-after-dhcp-lease-obtained/)
- [Proxmox 安装程序无法加载 GUI 安装程序](https://www.reddit.com/r/Proxmox/comments/130of40/proxmox_installer_fail_to_load_gui_installer/)

解决方法：

执行

```bash
chmod 1777 /tmp   
Xorg -configure   
mv /xorg.conf.new /etc/X11/xorg.conf
```

再执行

```bash
vim /etc/X11/xorg.conf
```

找到 "device" 一节中的 "Driver" 设置，将值修改为 "fbdev" 。

启动x 图形界面，就可以继续图形化安装 pve。

```bash
startx
```

但我遇到的问题是安装完成之后，又卡在 "Loading initial ramdisk" 了。

### "Loading initial ramdisk"

U 盘启动后，选择安装 PVE，然后在命令行界面上看到 "Loading initial ramdisk ......" 之后就进入黑屏状态，无任何显示。或者是屏幕保持不变，卡死在这个界面再也无法继续。

通过逐个排查 cmos 设置选项，最后发现是和 cpu 节能配置有关，必须将 "cpu C state control" 的这两个设置项设置为 Disable 才能避免出现这个问题。

- CPU C3 report
- CPU C6 report

这两个设置项如果在安装完成之后再开启，也会导致 PVE 启动时同样卡在 initial ramdisk 上，因此必须保证始终关闭。

备注：这个问题仅有某些机器(x99主板)上会出现，不是所有机器都有这个问题，比如z690主板上我发现就可以开启 C3 report 和 C6 report 之后继续安装和使用。

有人说是显卡的兼容性，可以通过使用集显来绕开这个问题：

- [Proxmox VE(PVE) 安装时卡顿在 loading initial ramdisk 的解决办法 - 古道轻风 - 博客园 (cnblogs.com)](https://www.cnblogs.com/88223100/p/PVE_loading-initial-ramdisk.html)

也有通过增加 nomodeset 参数来解决的： 

https://mengkai.fun/archives/1717297733491

我测试中发现（z690主板 + 13700kf cpu + nvidia 亮机卡）并不能解决问题： 1. 在Terminal UI界面通过增加参数 nomodeset 跳过这个报错之后，虽然安装成功，但是 pve 用不了。 2. 在 在 Graphic UI界面也可以增加参数 nomodeset，但随后的图形界面会有显示bug，无法正确显示。

最后，发现可能和 nvidia 显卡有关，因为出现报错:

```bash
loading dirvers: nvidiafb .......................
```

- https://www.reddit.com/r/Proxmox/comments/1cuvv2q/proxmox_821_installer_is_stuck_at_loading_some/?rdt=46925
- https://forum.proxmox.com/threads/installing-proxmox-8-1-crashing-when-loading-nvidiafb-driver.143363/

解决方法有两个：

1. 如果有集成显卡，请使用集成显卡进行安装，这样就不会载入 nvidiafb
2. 可以换成 amd 显卡

注意这个报错只出现在 pve 安装时，安装完成后 nvidia 显卡时可以使用的。因此有一个绕开这个问题的做法就是：先用集成显卡或者 amd 独立显卡完成 pve 的安装之后，再换成 nvidia 显卡。

### "create LVs"

安装时，有时会遇到长时间（三五分钟）停留在 "create LVs" 处，原因不明，大部分机器没这个问题。

耐心等待就好，暂时不清楚问题，好在只是慢。

### nvme device not ready

和具体使用的 nvme ssd 有关，我用的爱国者 p7000z 4t 版本就有这个问题，安装正常，但是启动 PVE 时会报错：

```
nvme nvme0: Device not ready; aborting initialisation, CSTS=0x0
```

参考帖子：

- [**联芸MAP1602主控的可以入了，掉坑里刚爬出来，P7000Z晚班车拿了四块，附内核**](https://www.chiphell.com/forum.php?mod=viewthread&tid=2524660&extra=page%3D1&ordertype=1&mobile=no)

备注：pve8.1 版本之后就没有这个问题了，解决方案已经直接进入了新的 linux 内核。