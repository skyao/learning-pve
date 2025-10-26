---
title: "固定网卡名称"
linkTitle: "固定网卡名称"
weight: 80
date: 2025-04-14
description: >
  固定网卡名称避免因pcie顺序发生变化造成网卡名称不一致
---

在Proxmox VE中，当 PCIe 设备发生变化时，网络接口名称可能会重新排序，这是一个已知问题。要解决这个问题，可以通过创建持久的网络接口命名规则来固定网卡名称。

## 问题描述

执行 

```bash
ip addr
```

可以看到：

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
3: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master vmbr0 state UP group default qlen 1000
    link/ether 40:8d:5c:b4:9b:e2 brd ff:ff:ff:ff:ff:ff
    altname enp0s25
4: ens4f0np0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq master vmbr0 state DOWN group default qlen 1000
    link/ether 0c:42:a1:62:1a:88 brd ff:ff:ff:ff:ff:ff
    altname enp5s0f0np0
5: ens4f1np1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq master vmbr0 state DOWN group default qlen 1000
    link/ether 0c:42:a1:62:1a:89 brd ff:ff:ff:ff:ff:ff
    altname enp5s0f1np1
6: enp6s0f0np0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq master vmbr0 state DOWN group default qlen 1000
    link/ether 04:3f:72:c5:7e:fa brd ff:ff:ff:ff:ff:ff
7: enp6s0f1np1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq master vmbr0 state DOWN group default qlen 1000
    link/ether 04:3f:72:c5:7e:fb brd ff:ff:ff:ff:ff:ff
8: vmbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 40:8d:5c:b4:9b:e2 brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.98/24 scope global vmbr0
       valid_lft forever preferred_lft forever
    inet6 fe80::428d:5cff:feb4:9be2/64 scope link
       valid_lft forever preferred_lft forever
9: tap1000i0: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master fwbr1000i0 state UNKNOWN group default qlen 1000
    link/ether 2a:1d:4a:70:6d:93 brd ff:ff:ff:ff:ff:ff
10: fwbr1000i0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 9e:75:1d:6a:2a:bd brd ff:ff:ff:ff:ff:ff
11: fwpr1000p0@fwln1000i0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master vmbr0 state UP group default qlen 1000
    link/ether 42:c3:f0:25:de:17 brd ff:ff:ff:ff:ff:ff
12: fwln1000i0@fwpr1000p0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master fwbr1000i0 state UP group default qlen 1000
    link/ether 9e:75:1d:6a:2a:bd brd ff:ff:ff:ff:ff:ff
13: tap9002i0: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master vmbr0 state UNKNOWN group default qlen 1000
    link/ether 4e:da:a7:3c:4c:92 brd ff:ff:ff:ff:ff:ff
```

去掉 lo 和虚拟网卡，只剩下：

```bash
3: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master vmbr0 state UP group default qlen 1000
    link/ether 40:8d:5c:b4:9b:e2 brd ff:ff:ff:ff:ff:ff
    altname enp0s25
4: ens4f0np0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq master vmbr0 state DOWN group default qlen 1000
    link/ether 0c:42:a1:62:1a:88 brd ff:ff:ff:ff:ff:ff
    altname enp5s0f0np0
5: ens4f1np1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq master vmbr0 state DOWN group default qlen 1000
    link/ether 0c:42:a1:62:1a:89 brd ff:ff:ff:ff:ff:ff
    altname enp5s0f1np1
6: enp6s0f0np0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq master vmbr0 state DOWN group default qlen 1000
    link/ether 04:3f:72:c5:7e:fa brd ff:ff:ff:ff:ff:ff
7: enp6s0f1np1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq master vmbr0 state DOWN group default qlen 1000
    link/ether 04:3f:72:c5:7e:fb brd ff:ff:ff:ff:ff:ff
8: vmbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 40:8d:5c:b4:9b:e2 brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.98/24 scope global vmbr0
       valid_lft forever preferred_lft forever
    inet6 fe80::428d:5cff:feb4:9be2/64 scope link
       valid_lft forever preferred_lft forever

```

为了让 linux bridge 工作, vmbr0 的 bridge-ports 就必须将所有的网卡名称列进去:

```bash
vi /etc/network/interfaces
```

可以看到 bridge-ports：

```bash
auto vmbr0
iface vmbr0 inet static
        address 192.168.3.98/24
        gateway 192.168.3.1
        bridge-ports eno1 ens4f0np0 ens4f1np1 enp6s0f0np0 enp6s0f1np1
        bridge-stp off
        bridge-fd 0
```

网卡的命名中有数字表示顺序，这个顺序取决于系统内所有 pcie 设备的排序，例如：

```bash
$ lspci

00:00.0 Host bridge: Intel Corporation Xeon E7 v4/Xeon E5 v4/Xeon E3 v4/Xeon D DMI2 (rev 01)
00:01.0 PCI bridge: Intel Corporation Xeon E7 v4/Xeon E5 v4/Xeon E3 v4/Xeon D PCI Express Root Port 1 (rev 01)
00:02.0 PCI bridge: Intel Corporation Xeon E7 v4/Xeon E5 v4/Xeon E3 v4/Xeon D PCI Express Root Port 2 (rev 01)
00:02.2 PCI bridge: Intel Corporation Xeon E7 v4/Xeon E5 v4/Xeon E3 v4/Xeon D PCI Express Root Port 2 (rev 01)
00:02.3 PCI bridge: Intel Corporation Xeon E7 v4/Xeon E5 v4/Xeon E3 v4/Xeon D PCI Express Root Port 2 (rev 01)
00:03.0 PCI bridge: Intel Corporation Xeon E7 v4/Xeon E5 v4/Xeon E3 v4/Xeon D PCI Express Root Port 3 (rev 01)
00:03.2 PCI bridge: Intel Corporation Xeon E7 v4/Xeon E5 v4/Xeon E3 v4/Xeon D PCI Express Root Port 3 (rev 01)
00:05.0 System peripheral: Intel Corporation Xeon E7 v4/Xeon E5 v4/Xeon E3 v4/Xeon D Map/VTd_Misc/System Management (rev 01)
00:05.1 System peripheral: Intel Corporation Xeon E7 v4/Xeon E5 v4/Xeon E3 v4/Xeon D IIO Hot Plug (rev 01)
00:05.2 System peripheral: Intel Corporation Xeon E7 v4/Xeon E5 v4/Xeon E3 v4/Xeon D IIO RAS/Control Status/Global Errors (rev 01)
00:11.0 Unassigned class [ff00]: Intel Corporation C610/X99 series chipset SPSR (rev 05)
00:14.0 USB controller: Intel Corporation C610/X99 series chipset USB xHCI Host Controller (rev 05)
00:16.0 Communication controller: Intel Corporation C610/X99 series chipset MEI Controller #1 (rev 05)
00:19.0 Ethernet controller: Intel Corporation Ethernet Connection (2) I218-V (rev 05)
00:1a.0 USB controller: Intel Corporation C610/X99 series chipset USB Enhanced Host Controller #2 (rev 05)
00:1b.0 Audio device: Intel Corporation C610/X99 series chipset HD Audio Controller (rev 05)
00:1c.0 PCI bridge: Intel Corporation C610/X99 series chipset PCI Express Root Port #1 (rev d5)
00:1c.5 PCI bridge: Intel Corporation C610/X99 series chipset PCI Express Root Port #6 (rev d5)
00:1c.7 PCI bridge: Intel Corporation C610/X99 series chipset PCI Express Root Port #8 (rev d5)
00:1d.0 USB controller: Intel Corporation C610/X99 series chipset USB Enhanced Host Controller #1 (rev 05)
00:1f.0 ISA bridge: Intel Corporation C610/X99 series chipset LPC Controller (rev 05)
00:1f.2 SATA controller: Intel Corporation C610/X99 series chipset 6-Port SATA Controller [AHCI mode] (rev 05)
00:1f.3 SMBus: Intel Corporation C610/X99 series chipset SMBus Controller (rev 05)
03:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981/PM983
04:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981/PM983
05:00.0 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
05:00.1 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
06:00.0 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
06:00.1 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
07:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981/PM983
08:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8125 2.5GbE Controller (rev 05)
09:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Oland [Radeon HD 8570 / R5 430 OEM / R7 240/340 / Radeon 520 OEM]
09:00.1 Audio device: Advanced Micro Devices, Inc. [AMD/ATI] Oland/Hainan/Cape Verde/Pitcairn HDMI Audio [Radeon HD 7000 Series]
......
```

单独看网卡是这样的，我这里有四块网卡，其中两块25g cx4121 网卡是双头的：

```bash
$ lspci | grep Ethernet

00:19.0 Ethernet controller: Intel Corporation Ethernet Connection (2) I218-V (rev 05)
05:00.0 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
05:00.1 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
06:00.0 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
06:00.1 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
08:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8125 2.5GbE Controller (rev 05)
```

可以用下面的命令查看：

```bash
$ udevadm info -a /sys/class/net/enp6s0f1np1 | grep "KERNELS==" | head -n 1
    KERNELS=="0000:06:00.1"
```

输出的 `KERNELS=="0000:06:00.1"` 就是这块网卡的 pcie 顺序。然后根据这个顺序推导出网卡名称 enp6s0f1np1。

备注：这里的 08:00.0 这块网卡被我直通给 openwrt 虚拟机了，因此不放在 vmbr0 中的 bridge-ports。

因此，一旦 pcie 设备有变化，比如增加或者减少了某个设备（不限于网卡，也可以是其他设备），比如开启或者关闭了板载声卡之类，也会导致 pcie 顺序的变化，从而造成网卡名称的变化，最后基于网卡名称的各种设置就都会发生错乱， 比如 vmbr0 的 bridge-ports。

## 解决问题的思路

基本思路就是在网卡命名时，不要绑定不稳定的 pcie 顺序，而且绑定其他固定的属性，最常用的方案就是绑定 MAC 地址。

通常情况下，一个 pve 设备中的网卡数量和网卡 mac 地址都是固定的，除非明确更换或者增减网卡，否则不会发生变化。

这样，不管网卡和其他 pcie 设备的增减和顺序调整，都不会影响到网卡的命名，然后基于网卡名称的各种设备也就稳定了。

因此，后面将根据网卡的 mac 地址，将网卡重新命名为，暂时定义的规则为类似 en25g1 这样：

- en：前缀，表示 Ethernet 以太网网卡
- 1g：网卡速度，根据实际情况可能有 1g/2g/10g/40g/100g，分别对应千兆，2.5g网卡，万兆，40g，100g网卡
- 1：网卡序号，在同速度网卡中排列，方便起见，不以 0 开头，而是 1/2/3/4 这样表示第一/第二/第三/第四块网卡

按照这样的规则，上面的五块网卡，名称会固定为

- en1g1
- en25g1 / en25g2 / en25g3 / en25g4

这样 vmbr0 的 bridge-ports 就可以写成 `en1g1 en25g1 en25g2 en25g3 en25g4`，之后不管网卡顺序和pcie设备调整，都不会影响 vmbr0 的 bridge-ports 工作。

## 实现方案(pve9)

注意：在 Proxmox VE 9（基于 Debian 13）中，固定网卡名称的方式与 PVE 8（基于 Debian 12）有所不同，主要是因为 Debian 13 对 udev 规则的处理逻辑做了调整，传统的 70-persistent-net.rules 方式可能不再生效。

在 PVE 9 中，推荐通过 systemd 的 .link 配置文件 来固定网卡名称，这是更现代且兼容的方式。

步骤如下：

1. 找出需要固定的网卡 MAC 地址

     ```bash
     $ ip addr

     2: enp1s0np0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master vmbr0 state UP group default qlen 1000
     link/ether 8c:4a:8e:88:7d:5a brd ff:ff:ff:ff:ff:ff
     ```

2. 创建 .link 配置文件

     ```bash
     vi /etc/systemd/network/10-static-enp1s0np0.link
     ```

     内容为：

     ```properties
     [Match]
     MACAddress=8c:3a:8e:88:7d:5a

     [Link]
     Name=en100g1
     ```

3. 修改 vmbr0 的配置

     因为一般都是远程操作，为了安全起见，重命名前后的两个网卡名字都加入到 vmbr0，以防万一出错时无法连接管理网络：

     ```bash
     vi /etc/network/interfaces
     ```

     内容为：

     ```properties
     iface enp1s0np0 inet manual
     iface en100g1 inet manual
     
     auto vmbr0
     iface vmbr0 inet static
            ......
            bridge-ports enp1s0np0 en100g1
    ```

4. 重启

     ```bash
     reboot
     ```

     之后查看网卡名称发现已经固定：

     ```bash
     $ ip addr

     2: en100g1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master vmbr0 state UP group default qlen 1000
     link/ether 8c:3a:8e:88:7d:5a brd ff:ff:ff:ff:ff:ff
     altname enx8c2a8e886d5a
     ```

5. 修复 vmbr0 的 bridge-ports，把改名之前的网卡名字去掉。

对于只有多个网卡的情况，操作要复杂一下，主要是需要区分是哪些网卡。如我这里有一台机器上有 100g 和 25g 两块双头网卡， 提供四个网口，其中 100g 网卡提供两个网口，25g 网卡提供两个网口。

需要通过下面两个命令找出哪些网卡是 100g 网卡，哪些网卡是 25g 网卡， 记录好 mac 地址，然后再逐个网口进行操作：

```bash
ip addr
lspci | grep Ethernet

vi /etc/systemd/network/10-static-en25g1.link
vi /etc/systemd/network/10-static-en25g2.link
vi /etc/systemd/network/10-static-en100g1.link
vi /etc/systemd/network/10-static-en100g2.link
```

## 实现方案(pve8)

### 准备脚本

```bash
mkdir -p ~/work/soft/pve
cd ~/work/soft/pve

vi ~/work/soft/pve/fix_pve_nic_name.sh 
```

输入以下内容：

```bash
#!/bin/bash

# 检查 root 权限
if [ "$(id -u)" -ne 0 ]; then
    echo "请使用 root 用户运行此脚本！" >&2
    exit 1
fi

RULE_FILE="/etc/udev/rules.d/70-persistent-net.rules"

echo "正在生成基于 MAC 地址的网卡命名规则..."
echo "# 由脚本自动生成的持久化网卡命名规则（基于MAC地址）" > "$RULE_FILE"
echo "# 生成时间: $(date)" >> "$RULE_FILE"
echo "" >> "$RULE_FILE"

# 遍历所有网络接口
for DEV in /sys/class/net/*; do
    INTERFACE=$(basename "$DEV")
    MAC_FILE="$DEV/address"

    # 跳过虚拟接口和没有MAC地址的设备
    [[ "$INTERFACE" == "lo" ]] && continue
    [[ ! -f "$MAC_FILE" ]] && continue

    MAC=$(cat "$MAC_FILE" 2>/dev/null)
    [[ -z "$MAC" ]] && continue

    # 写入规则（仅使用 MAC 地址）
    echo "SUBSYSTEM==\"net\", ACTION==\"add\", ATTR{address}==\"$MAC\", NAME=\"$INTERFACE\"" >> "$RULE_FILE"
done

# 重新加载 udev 规则
# udevadm control --reload-rules
# udevadm trigger

echo ""
echo "规则已生成到 $RULE_FILE，内容如下："
echo "----------------------------------------"
cat "$RULE_FILE"
echo "----------------------------------------"
echo ""
```

然后添加执行权限：

```bash
chmod +x fix_pve_nic_name.sh
```

再执行该脚本：

```bash
./fix_pve_nic_name.sh
```

输出为：

```bash
正在生成基于 MAC 地址的网卡命名规则...

规则已生成到 /etc/udev/rules.d/70-persistent-net.rules，内容如下：
----------------------------------------
# 由脚本自动生成的持久化网卡命名规则（基于MAC地址）
# 生成时间: Mon Apr 14 06:31:37 AM CST 2025

SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="40:8d:5c:b4:9b:e2", NAME="eno1"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="04:3f:72:c5:7e:fa", NAME="enp6s0f0np0"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="04:3f:72:c5:7e:fb", NAME="enp6s0f1np1"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="0c:42:a1:62:1a:88", NAME="ens4f0np0"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="0c:42:a1:62:1a:89", NAME="ens4f1np1"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="9e:75:1d:6a:2a:bd", NAME="fwbr1000i0"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="9e:75:1d:6a:2a:bd", NAME="fwln1000i0"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="42:c3:f0:25:de:17", NAME="fwpr1000p0"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="2a:1d:4a:70:6d:93", NAME="tap1000i0"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="4e:da:a7:3c:4c:92", NAME="tap9002i0"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="40:8d:5c:b4:9b:e2", NAME="vmbr0"
```

```bash
vi /etc/udev/rules.d/70-persistent-net.rules
```

打开 `/etc/udev/rules.d/70-persistent-net.rules` 文件，删除不需要的内容，只保留这物理网卡：

```properties
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="40:8d:5c:b4:9b:e2", NAME="eno1"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="04:3f:72:c5:7e:fa", NAME="enp6s0f0np0"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="04:3f:72:c5:7e:fb", NAME="enp6s0f1np1"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="0c:42:a1:62:1a:88", NAME="ens4f0np0"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="0c:42:a1:62:1a:89", NAME="ens4f1np1"
```

然后按照前面的命名规则修改网卡名字：

```properties
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="40:8d:5c:b4:9b:e2", NAME="en1g1"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="04:3f:72:c5:7e:fa", NAME="en25g3"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="04:3f:72:c5:7e:fb", NAME="en25g4"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="0c:42:a1:62:1a:88", NAME="en25g1"
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="0c:42:a1:62:1a:89", NAME="en25g2"
```

执行下面的命令让修改好的规则生效：

```bash
udevadm control --reload-rules
udevadm trigger
```

执行下面的命令检验一下重命名的规则：

```bash
udevadm test /sys/class/net/eno1 2>&1 | grep -i 'renamed'
udevadm test /sys/class/net/ens4f0np0 2>&1 | grep -i 'renamed'
udevadm test /sys/class/net/ens4f1np1 2>&1 | grep -i 'renamed'
udevadm test /sys/class/net/enp6s0f0np0 2>&1 | grep -i 'renamed'
udevadm test /sys/class/net/enp6s0f1np1 2>&1 | grep -i 'renamed'
```

输出为：

```bash
en1g1: Network interface 3 is renamed from 'eno1' to 'en1g1'
en25g1: Network interface 4 is renamed from 'ens4f0np0' to 'en25g1'
en25g2: Network interface 5 is renamed from 'ens4f1np1' to 'en25g2'
en25g3: Network interface 6 is renamed from 'enp6s0f0np0' to 'en25g3'
en25g4: Network interface 7 is renamed from 'enp6s0f1np1' to 'en25g4'
```

最后，去修改 vmbr0 的 bridge-ports：

```bash
# 修改前备份一下
cp /etc/network/interfaces /etc/network/interfaces_backup
vi /etc/network/interfaces
```

设置内容为：

```bash
auto lo
iface lo inet loopback

iface en1g1 inet manual

iface en25g1 inet manual

iface en25g2 inet manual

iface en25g3 inet manual

iface en25g4 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.3.98/24
        gateway 192.168.3.1
        bridge-ports en1g1 en25g1 en25g2 en25g3 en25g4
        bridge-stp off
        bridge-fd 0

source /etc/network/interfaces.d/*
```

重启 pve 机器。

之后重新登录，

```bash
ip addr
```

查看网卡信息:

```bash
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
3: en1g1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master vmbr0 state UP group default qlen 1000
    link/ether 40:8d:5c:b4:9b:e2 brd ff:ff:ff:ff:ff:ff
    altname enp0s25
    altname eno1
4: en25g1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq master vmbr0 state DOWN group default qlen 1000
    link/ether 0c:42:a1:62:1a:88 brd ff:ff:ff:ff:ff:ff
    altname enp5s0f0np0
    altname ens4f0np0
5: en25g2: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq master vmbr0 state DOWN group default qlen 1000
    link/ether 0c:42:a1:62:1a:89 brd ff:ff:ff:ff:ff:ff
    altname enp5s0f1np1
    altname ens4f1np1
6: en25g3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq master vmbr0 state DOWN group default qlen 1000
    link/ether 04:3f:72:c5:7e:fa brd ff:ff:ff:ff:ff:ff
    altname enp6s0f0np0
7: en25g4: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq master vmbr0 state DOWN group default qlen 1000
    link/ether 04:3f:72:c5:7e:fb brd ff:ff:ff:ff:ff:ff
    altname enp6s0f1np1
8: vmbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 40:8d:5c:b4:9b:e2 brd ff:ff:ff:ff:ff:ff
    inet 192.168.3.98/24 scope global vmbr0
       valid_lft forever preferred_lft forever
    inet6 fe80::428d:5cff:feb4:9be2/64 scope link
       valid_lft forever preferred_lft forever
```

至此网卡的名称就被固定下来，不受 pcie 设备顺序的影响了。

