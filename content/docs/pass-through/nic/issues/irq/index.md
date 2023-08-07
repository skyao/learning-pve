---
title: "IRQ冲突"
linkTitle: "IRQ冲突"
weight: 20
date: 2021-01-18
description: >
  直通网卡时因 IRQ 冲突造成不可用
---

## 故障描述

插入多块网卡后设置直通，在虚拟机启动时报错，错误描述如下：

```
kvm: -device vfio-pci,host=0000:01:00.0,id=hostpci1,bus=ich9-pcie-port-2,addr=0x0: vfio 0000:01:00.0: Failed to set up TRIGGER eventfd signaling for interrupt INTX-0: VFIO_DEVICE_SET_IRQS failure: Device or resource busy
TASK ERROR: start failed: QEMU exited with code 1
```



## 查找问题

### 检查 INTx disable

参考这个帖子的描述：

https://forums.unraid.net/topic/120895-passing-a-capture-card-to-vms/

可以用下面这个 check.sh 脚本来检查网卡是否设置了 INTx disable ：

```bash
#!/bin/sh
# Usage $0 <PCI device>, ex: 9:00.0

INTX=$(( 0x400 ))
ORIG=$(( 0x$(setpci -s $1 4.w) ))

if [ $(( $INTX & $ORIG )) -ne 0 ]; then
echo "INTx disable supported and enabled on $1"
exit 0
fi

NEW=$(printf %04x $(( $INTX | $ORIG )))
setpci -s $1 4.w=$NEW
NEW=$(( 0x$(setpci -s $1 4.w) ))

if [ $(( $INTX & $NEW )) -ne 0 ]; then
echo "INTx disable support available on $1"
else
echo "INTx disable support NOT available on $1"
fi

NEW=$(printf %04x $ORIG)
setpci -s $1 4.w=$NEW
```

执行结果如下：

```bash
➜  ~ ./check.sh 01:00.0
INTx disable support available on 01:00.0
➜  ~ ./check.sh 04:00.0
INTx disable support available on 04:00.0
➜  ~ ./check.sh 08:00.0
INTx disable support available on 08:00.0
➜  ~ ./check.sh 09:00.0
INTx disable supported and enabled on 09:00.0
```

### 检查启动日志

可以执行

```bash
dmesg
```

查看 pve 的启动日志，找到其中报错的内容：

```bash
[ 3852.066855] vfio-pci 0000:04:00.0: vfio_ecap_init: hiding ecap 0x19@0x18c
[ 3852.210816] vfio-pci 0000:01:00.0: vfio_ecap_init: hiding ecap 0x19@0x18c
[ 3852.219438] genirq: Flags mismatch irq 16. 00000000 (vfio-intx(0000:01:00.0)) vs. 00000000 (vfio-intx(0000:04:00.0))


 3940.230432] vfio-pci 0000:04:00.0: vfio_ecap_init: hiding ecap 0x19@0x18c
[ 3940.374391] vfio-pci 0000:08:00.0: vfio_ecap_init: hiding ecap 0x19@0x18c
[ 3940.383218] genirq: Flags mismatch irq 16. 00000000 (vfio-intx(0000:08:00.0)) vs. 00000000 (vfio-intx(0000:04:00.0))
```

### 查看 IRQ 情况

IRQ 情况可以这样查看：

```bash
cat /proc/interrupts
```

输出为：

```bash
            CPU0       CPU1       CPU2       CPU3       CPU4       CPU5       CPU6       CPU7       CPU8       CPU9       CPU10      CPU11      
   0:         11          0          0          0          0          0          0          0          0          0          0          0  IR-IO-APIC    2-edge      timer
   8:          0          0          0          0          0          0          0          0          0          0          0          0  IR-IO-APIC    8-edge      rtc0
   9:          0          0          0          0          0          0          0          0          0          0          0          0  IR-IO-APIC    9-fasteoi   acpi
  14:          0          0          0          0          0          0          0          0          0          0          0          0  IR-IO-APIC   14-fasteoi   INTC1056:00
  18:          0          0          0          5          0          0          0          0          0          0          0          0  IR-IO-APIC   18-fasteoi   i801_smbus
  27:          0          0          0          0          0          0          0          0          0          0          0          0  IR-IO-APIC   27-fasteoi   idma64.0, i2c_designware.0
  29:          0          0          0          0          0          0          0          0          0          0          0          0  IR-IO-APIC   29-fasteoi   idma64.2, i2c_designware.2
  40:          0          0          0          0          0          0          0          0          0          0          0          0  IR-IO-APIC   40-fasteoi   idma64.1, i2c_designware.1
 120:          0          0          0          0          0          0          0          0          0          0          0          0  DMAR-MSI    0-edge      dmar0
 121:          0          0          0          0          0          0          0          0          0          0          0          0  DMAR-MSI    1-edge      dmar1
 122:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSI-0000:00:01.0    0-edge      PCIe PME, aerdrv
 123:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSI-0000:00:1a.0    0-edge      PCIe PME, pciehp
 124:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSI-0000:00:1b.0    0-edge      PCIe PME
 125:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSI-0000:00:1b.4    0-edge      PCIe PME, aerdrv
 126:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSI-0000:00:1c.0    0-edge      PCIe PME, pciehp
 127:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSI-0000:00:1c.1    0-edge      PCIe PME, aerdrv
 128:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSI-0000:00:1c.2    0-edge      PCIe PME, aerdrv
 129:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSI-0000:00:1c.4    0-edge      PCIe PME, aerdrv
 130:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSI-0000:00:1d.0    0-edge      PCIe PME, aerdrv
 131:          0          0      13081          0          0          0          0          0          0          0          0          0  IR-PCI-MSI-0000:00:17.0    0-edge      ahci[0000:00:17.0]
 132:          0          0          0          0    1083661          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:06:00.0    0-edge      enp6s0
 133:          0          0          0          0         90          0          0          0          0          0          0          0  IR-PCI-MSI-0000:00:14.0    0-edge      xhci_hcd
 134:          0          0          0          0          0      88674          0          0          0          0          0          0  IR-PCI-MSIX-0000:07:00.0    0-edge      enp7s0
 210:          0          0          0          0          0          0          0          0          0          0          0         74  IR-PCI-MSIX-0000:09:00.0    0-edge      mlx4-async@pci:0000:09:00.0
 211:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0    1-edge      mlx4-1@0000:09:00.0
 212:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0    2-edge      mlx4-2@0000:09:00.0
 213:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0    3-edge      mlx4-3@0000:09:00.0
 214:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0    4-edge      mlx4-4@0000:09:00.0
 215:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0    5-edge      mlx4-5@0000:09:00.0
 216:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0    6-edge      mlx4-6@0000:09:00.0
 217:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0    7-edge      mlx4-7@0000:09:00.0
 218:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0    8-edge      mlx4-8@0000:09:00.0
 219:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0    9-edge      mlx4-9@0000:09:00.0
 220:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   10-edge      mlx4-10@0000:09:00.0
 221:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   11-edge      mlx4-11@0000:09:00.0
 222:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   12-edge      mlx4-12@0000:09:00.0
 223:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   13-edge      mlx4-13@0000:09:00.0
 224:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   14-edge      mlx4-14@0000:09:00.0
 225:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   15-edge      mlx4-15@0000:09:00.0
 226:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   16-edge      mlx4-16@0000:09:00.0
 227:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   17-edge      mlx4-17@0000:09:00.0
 228:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   18-edge      mlx4-18@0000:09:00.0
 229:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   19-edge      mlx4-19@0000:09:00.0
 230:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   20-edge      mlx4-20@0000:09:00.0
 231:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   21-edge      mlx4-21@0000:09:00.0
 232:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   22-edge      mlx4-22@0000:09:00.0
 233:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   23-edge      mlx4-23@0000:09:00.0
 234:          0          0          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSIX-0000:09:00.0   24-edge      mlx4-24@0000:09:00.0
 235:          0         49          0          0          0          0          0          0          0          0          0          0  IR-PCI-MSI-0000:00:16.0    0-edge      mei_me
 236:          0          0        271          0          0          0          0          0          0          0          0          0  IR-PCI-MSI-0000:00:02.0    0-edge      i915
 237:          0          0          0        750          0          0          0          0          0          0          0          0  IR-PCI-MSI-0000:00:1f.3    0-edge      snd_hda_intel:card0
 NMI:          1          0          3          0          5          1          1          6          1          0          1          0   Non-maskable interrupts
 LOC:      99118      43626     176733      55708     828067     207665     115311     154475      78857      10037      61928      11351   Local timer interrupts
 SPU:          0          0          0          0          0          0          0          0          0          0          0          0   Spurious interrupts
 PMI:          1          0          3          0          5          1          1          6          1          0          1          0   Performance monitoring interrupts
 IWI:      86064      28486     141932      40754     157189      94910      89143     116144      60778       5779      51049       5784   IRQ work interrupts
 RTR:          0          0          0          0          0          0          0          0          0          0          0          0   APIC ICR read retries
 RES:        310        314       5048        112        755        544        799       3192        584         60        481         99   Rescheduling interrupts
 CAL:      20663      17009      19217      16776      19075      18387      15627      15893      17333      18026      18963      17581   Function call interrupts
 TLB:         87         35        141         33         81         26         79         62         86         18         75          8   TLB shootdowns
 TRM:          1          1          1          1          1          1          1          1          1          1          1          1   Thermal event interrupts
 THR:          0          0          0          0          0          0          0          0          0          0          0          0   Threshold APIC interrupts
 DFR:          0          0          0          0          0          0          0          0          0          0          0          0   Deferred Error APIC interrupts
 MCE:          0          0          0          0          0          0          0          0          0          0          0          0   Machine check exceptions
 MCP:         17         18         18         18         18         18         18         18         18         18         18         18   Machine check polls
 ERR:          0
 MIS:          0
 PIN:          0          0          0          0          0          0          0          0          0          0          0          0   Posted-interrupt notification event
 NPI:          0          0          0          0          0          0          0          0          0          0          0          0   Nested posted-interrupt event
 PIW:          0          0          0          0          0          0          0          0          0          0          0          0   Posted-interrupt wakeup event
```

对于具体的网卡，可以通过 

```bash
 lspci -v -s 0000:01:00
```

来查看：

```bash
01:00.0 Ethernet controller: Mellanox Technologies MT27520 Family [ConnectX-3 Pro]
	Subsystem: Hewlett-Packard Company InfiniBand FDR/Ethernet 10Gb/40Gb 2-port 544+FLR-QSFP Adapter
	Flags: fast devsel, IRQ 16, IOMMU group 17
	Memory at 82200000 (64-bit, non-prefetchable) [disabled] [size=1M]
	Memory at 6166000000 (64-bit, prefetchable) [disabled] [size=32M]
	Capabilities: [40] Power Management version 3
	Capabilities: [48] Vital Product Data
	Capabilities: [9c] MSI-X: Enable- Count=128 Masked-
	Capabilities: [60] Express Endpoint, MSI 00
	Capabilities: [c0] Vendor Specific Information: Len=18 <?>
	Capabilities: [100] Alternative Routing-ID Interpretation (ARI)
	Capabilities: [148] Device Serial Number c4-34-6b-ff-ff-df-e1-80
	Capabilities: [108] Single Root I/O Virtualization (SR-IOV)
	Capabilities: [154] Advanced Error Reporting
	Capabilities: [18c] Secondary PCI Express
	Kernel driver in use: vfio-pci
	Kernel modules: mlx4_core
```

IRQ 16，后面发现另外一个网卡（hp561万兆电口网卡）同样使用 IRQ 16 从而造成冲突。



## 解决方案

### ~~删除造成 irq 冲突的硬件~~

可以用如下方法来删除造成冲突的硬件 ：

```bash
echo -n 1 > "/sys/devices/pci0000:00/0000:00:1b.0/remove"
```

通过 lspci 可以看到设备 device id ：

```bash
lspci

00:00.0 Host bridge: Intel Corporation Device 4650 (rev 05)
00:01.0 PCI bridge: Intel Corporation 12th Gen Core Processor PCI Express x16 Controller #1 (rev 05)
00:02.0 VGA compatible controller: Intel Corporation Alder Lake-S GT1 [UHD Graphics 730] (rev 0c)
00:0a.0 Signal processing controller: Intel Corporation Platform Monitoring Technology (rev 01)
00:14.0 USB controller: Intel Corporation Alder Lake-S PCH USB 3.2 Gen 2x2 XHCI Controller (rev 11)
00:14.2 RAM memory: Intel Corporation Alder Lake-S PCH Shared SRAM (rev 11)
00:15.0 Serial bus controller: Intel Corporation Alder Lake-S PCH Serial IO I2C Controller #0 (rev 11)
00:15.1 Serial bus controller: Intel Corporation Alder Lake-S PCH Serial IO I2C Controller #1 (rev 11)
00:15.2 Serial bus controller: Intel Corporation Alder Lake-S PCH Serial IO I2C Controller #2 (rev 11)
00:16.0 Communication controller: Intel Corporation Alder Lake-S PCH HECI Controller #1 (rev 11)
00:17.0 SATA controller: Intel Corporation Alder Lake-S PCH SATA Controller [AHCI Mode] (rev 11)
00:1a.0 PCI bridge: Intel Corporation Alder Lake-S PCH PCI Express Root Port #25 (rev 11)
00:1b.0 PCI bridge: Intel Corporation Device 7ac0 (rev 11)
00:1b.4 PCI bridge: Intel Corporation Device 7ac4 (rev 11)
00:1c.0 PCI bridge: Intel Corporation Alder Lake-S PCH PCI Express Root Port #1 (rev 11)
00:1c.1 PCI bridge: Intel Corporation Alder Lake-S PCH PCI Express Root Port #2 (rev 11)
00:1c.2 PCI bridge: Intel Corporation Device 7aba (rev 11)
00:1c.4 PCI bridge: Intel Corporation Alder Lake-S PCH PCI Express Root Port #5 (rev 11)
00:1d.0 PCI bridge: Intel Corporation Alder Lake-S PCH PCI Express Root Port #9 (rev 11)
00:1f.0 ISA bridge: Intel Corporation Z690 Chipset LPC/eSPI Controller (rev 11)
00:1f.3 Audio device: Intel Corporation Alder Lake-S HD Audio Controller (rev 11)
00:1f.4 SMBus: Intel Corporation Alder Lake-S PCH SMBus Controller (rev 11)
00:1f.5 Serial bus controller: Intel Corporation Alder Lake-S PCH SPI Controller (rev 11)

01:00.0 Ethernet controller: Mellanox Technologies MT27520 Family [ConnectX-3 Pro]
04:00.0 Ethernet controller: Mellanox Technologies MT27520 Family [ConnectX-3 Pro]
06:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller (rev 02)
07:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8125 2.5GbE Controller (rev 05)
08:00.0 Ethernet controller: Mellanox Technologies MT27520 Family [ConnectX-3 Pro]
09:00.0 Ethernet controller: Mellanox Technologies MT27520 Family [ConnectX-3 Pro]
```

尝试过，如下设置，无效：

```bash
mkdir -p /sys/devices/pci0000:00/0000:01:00.0/
mkdir -p /sys/devices/pci0000:00/0000:04:00.0/
mkdir -p /sys/devices/pci0000:00/0000:08:00.0/
mkdir -p /sys/devices/pci0000:00/0000:09:00.0/
echo -n 1 > "/sys/devices/pci0000:00/0000:01:00.0/remove"
echo -n 1 > "/sys/devices/pci0000:00/0000:04:00.0/remove"
echo -n 1 > "/sys/devices/pci0000:00/0000:08:00.0/remove"
echo -n 1 > "/sys/devices/pci0000:00/0000:09:00.0/remove"
```

## 后记

这个问题第一次在 华硕 z690-p 主板上遇到，当时插了三块 hp544+ 和一块 hp561，结果 hp561 和 hp544+ 冲突了。

后来在 z87 主板上遇到类似的问题，IRQ 冲突严重到 pve 系统都不稳定。不得已只好放弃在这两个主板上插多块网卡的想法。