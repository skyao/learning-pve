---
title: "清理不再使用的内核"
linkTitle: "清理内核"
weight: 35
date: 2021-01-18
description: >
  清理不再使用的内核
---

{{% alert title="严重警告" color="warning" %}} 

清理内核是一个高危操作，一旦失误系统无法启动，也无法远程救援。

**清理之前一定要确认已经重启**，先重启再清理，先重启再清理，先重启再清理！

非必要不远程操作！
{{% /alert %}}

## 查看现有内核

日常排查组合:

```bash
echo "=== 当前运行内核 ==="
uname -r
echo -e "\n=== dpkg已安装内核包 ==="
dpkg -l | grep proxmox-kernel
echo -e "\n=== /boot内核镜像 ==="
ls -lh /boot/vmlinuz-*
echo -e "\n=== /lib/modules模块目录 ==="
ls /lib/modules/
```

输出例如：

```bash
=== 当前运行内核 ===
7.0.14-12-pve

=== dpkg已安装内核包 ===
ii  proxmox-kernel-7.0                   7.0.14-12                            amd64        Latest Proxmox Kernel Image
ii  proxmox-kernel-7.0.14-12-pve         7.0.14-12                            amd64        Proxmox Kernel Image
ii  proxmox-kernel-7.0.2-6-pve-signed    7.0.2-6                              amd64        Proxmox Kernel Image (signed)
ii  proxmox-kernel-helper                9.2.0                                all          Function for various kernel maintenance tasks.

=== /boot内核镜像 ===
-rw-r--r-- 1 root root 16M Aug 11 19:05 /boot/vmlinuz-7.0.14-12-pve
-rw-r--r-- 1 root root 16M May 20 16:55 /boot/vmlinuz-7.0.2-6-pve

=== /lib/modules模块目录 ===
7.0.14-12-pve  7.0.2-6-pve
```

解释，查看当前**正在运行**的内核（开机后正在用的）：

```bash
$ uname -r

7.0.14-12-pve
```

查看 dpkg 数据库里**已经安装**的内核包（系统识别的全部内核）:

```bash
$ dpkg -l | grep proxmox-kernel

ii  proxmox-kernel-7.0                   7.0.14-12                            amd64        Latest Proxmox Kernel Image
ii  proxmox-kernel-7.0.14-12-pve         7.0.14-12                            amd64        Proxmox Kernel Image
ii  proxmox-kernel-7.0.2-6-pve-signed    7.0.2-6                              amd64        Proxmox Kernel Image (signed)
ii  proxmox-kernel-helper                9.2.0                                all          Function for various kernel maintenance tasks.
```

/boot 目录实际存在的内核镜像（磁盘上真实存在的 vmlinuz/initrd）:

```bash
$ ls -lh /boot/vmlinuz-* /boot/initrd.img-*

-rw-r--r-- 1 root root 88M Aug 16 18:16 /boot/initrd.img-7.0.14-12-pve
-rw-r--r-- 1 root root 88M Aug 16 08:56 /boot/initrd.img-7.0.2-6-pve
-rw-r--r-- 1 root root 16M Aug 11 19:05 /boot/vmlinuz-7.0.14-12-pve
-rw-r--r-- 1 root root 16M May 20 16:55 /boot/vmlinuz-7.0.2-6-pve
```

/lib/modules 已存在内核模块目录:

```bash
$ ls /lib/modules/

7.0.14-12-pve  7.0.2-6-pve
```

## 清理不用的内核

{{% alert title="警告" color="warning" %}} 

尽量保留两个内核，即正常使用的内核和一个备用的内核。

当前使用的内核损坏时，至少可以通过备用内核启动，方便修复。

{{% /alert %}}

### 脚本操作

https://tteck.github.io/Proxmox/

找到 Proxmox VE Kernel Clean 这个脚本，执行:

```bash
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/kernel-clean.sh)"
```

也可以手工把这个脚本下载到本地，方便以后执行本地运行：

```bash
mkdir -p ~/work/soft/pve
cd ~/work/soft/pve
wget https://github.com/tteck/Proxmox/raw/main/misc/kernel-clean.sh
chmod +x kernel-clean.sh
```

以后运行时，就只要执行

```bash
~/work/soft/pve/kernel-clean.sh
```

## 问题修复

### 未重启就删除新内核

一般发生在 apt upgrade 之后，新内核被安装并完成 grup 更新，成为默认内核。然后忘了重启，就用清理内核的脚本将新内核清理掉了。重启之后，因为默认启动内核被修改为新内核，而新内核文件被误删，因此 pve 启动失败。

修复的方式，只能接上显示器和键盘，在开机后 pve 的启动菜单中选择高级选项，在列出来的内核列表中一般会有新旧两个内核。选择旧内核先启动到 pve 中。

例如，假设操作涉及到的新旧内核分别为：

- pve-kernel-7.0.14-12-pve
- pve-kernel-7.0.2-6-pve

下面开始执行被误删的新内核的清理工作：

1. 清除 dpkg 残留记录

   ```bash
   dpkg --purge pve-kernel-7.0.14-12-pve
   ```

2. 手动清理磁盘上残余碎片

   ```bash
   # 检查 boot 目录有没有残缺文件：
   ls -lh /boot/*7.0.14-12-pve*

   # 如果还有残留 vmlinuz、initrd，直接删掉：
   rm -f /boot/*7.0.14-12-pve*

   # 检查 modules 目录残留：
   ls /lib/modules/ |grep 7.0.14-12
   # 如果目录还存在，删除
   rm -rf /lib/modules/7.0.14-12-pve
   ```

3. 更新 grub，把错误内核从引导列表清理掉

   ```bash
   update-grub
   proxmox-boot-tool refresh
   ```

   备注： PVE 使用`proxmox‑boot‑tool`管理 EFI/boot 分区，**必须执行，只跑 update‑grub 不够**。

重启，此时 7.0.14-12 已经被清理干净，正常会默认用 7.0.2-6 启动。

重新安装 7.0.14-12，此时 apt update 命令无效：

```bash
$ apt install pve-kernel-7.0.14-12-pve

Note, selecting 'proxmox-kernel-7.0.14-12-pve' instead of 'pve-kernel-7.0.14-12-pve'
proxmox-kernel-7.0.14-12-pve is already the newest version (7.0.14-12).
proxmox-kernel-7.0.14-12-pve set to manually installed.
Summary:
  Upgrading: 0, Installing: 0, Removing: 0, Not Upgrading: 0
```

需要用 apt reinstall ：

```bash
apt reinstall proxmox-kernel-7.0.14-12-pve
```

安装完成之后，立即重启，此时会默认用新的 7.0.14-12 内核启动。修复完成！

