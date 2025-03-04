---
title: "开启直通"
linkTitle: "开启直通"
weight: 10
date: 2021-01-18
description: >
  开启直通功能
---

可以通过两种方式开启直通功能：

1. 手工修改
2. 用 pvetools 工具帮忙

## 准备工作

### 主板bios设置



必须先在主板 bios 中开启 cpu 的 VD-T 支持和虚拟化支持。

VD-T不支持就无法直通。intel 要 b75 以上芯片组才支持。也就是需要从 intel 4代酷睿处理器开始才支持。

### 虚拟机设置

在创建虚拟机时，芯片组一定要**q35** 。因为Q35，才能 PCIE 直通，否则就是 PCI 直通。


## 手工

### 开启 iommu

```bash
vi /etc/default/grub
```

修改 GRUB_CMDLINE_LINUX_DEFAULT，增加 intel_iommu=on 。使用 intel_iommu=on 后，开机会有内核crash，可以尝试加一个 iommu=pt。这样就是：

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_pstate=disable intel_iommu=on iommu=pt pcie_acs_override=downstream pci=realloc=off"
```

> 备注： 安装完成后的默认值是 `GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_pstate=disable"`

解释：

- `pcie_acs_override=downstream`
-  `pci=realloc=off` 

修改完成之后，更新 grub：

```bash
update-grub
```

### 设置虚拟化驱动

```bash
vi /etc/modules
```

增加以下内容：

```properties
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

备注： 安装完成后的默认值是空，仅有少量注释。

升级并更新配置：

```bash
update-initramfs -u -k all
```

之后重启机器。


## pvetools

使用 pvetools 就简单了，只要在 pvetools 的菜单中选择 "配置PCI硬件直通" -》"配置开启物理机硬件直通支持"。

完成后重启机器。

## 检验

### 检验 iommu 是否开启

```bash
dmesg | grep -e DMAR -e IOMMU
```

正确开启时的输出会类似如下：

```bash
[    0.000000] Warning: PCIe ACS overrides enabled; This may allow non-IOMMU protected peer-to-peer DMA
[    0.033632] ACPI: DMAR 0x00000000BC4B0870 0000B8 (v01 INTEL  HSW      00000001 INTL 00000001)
[    0.033687] ACPI: Reserving DMAR table memory at [mem 0xbc4b0870-0xbc4b0927]
[    0.066765] DMAR: IOMMU enabled
[    0.186968] DMAR: Host address width 39
[    0.186972] DMAR: DRHD base: 0x000000fed90000 flags: 0x0
[    0.186988] DMAR: dmar0: reg_base_addr fed90000 ver 1:0 cap c0000020660462 ecap f0101a
[    0.186996] DMAR: DRHD base: 0x000000fed91000 flags: 0x1
[    0.187006] DMAR: dmar1: reg_base_addr fed91000 ver 1:0 cap d2008020660462 ecap f010da
[    0.187012] DMAR: RMRR base: 0x000000bc1d8000 end: 0x000000bc1e4fff
[    0.187017] DMAR: RMRR base: 0x000000bf000000 end: 0x000000cf1fffff
[    0.187024] DMAR-IR: IOAPIC id 8 under DRHD base  0xfed91000 IOMMU 1
[    0.187030] DMAR-IR: HPET id 0 under DRHD base 0xfed91000
[    0.187034] DMAR-IR: Queued invalidation will be enabled to support x2apic and Intr-remapping.
[    0.188070] DMAR-IR: Enabled IRQ remapping in x2apic mode
[    0.634998] DMAR: No ATSR found
[    0.635001] DMAR: No SATC found
[    0.635004] DMAR: IOMMU feature pgsel_inv inconsistent
[    0.635008] DMAR: IOMMU feature sc_support inconsistent
[    0.635011] DMAR: IOMMU feature pass_through inconsistent
[    0.635014] DMAR: dmar0: Using Queued invalidation
[    0.635026] DMAR: dmar1: Using Queued invalidation
[    0.720415] DMAR: Intel(R) Virtualization Technology for Directed I/O
[   14.512740] i915 0000:00:02.0: [drm] DMAR active, disabling use of stolen memory
```

可以看到 "DMAR: IOMMU enabled" / “DMAR: Intel(R) Virtualization Technology for Directed I/O” 的字样，说明 IOMMU 开启成功。

执行命令：

```bash
dmesg | grep 'remapping'
```

如果看到类似如下 “Enabled IRQ remapping in x2apic mode” 内容，也说明 IOMMU 开启成功：

```bash
[    0.187034] DMAR-IR: Queued invalidation will be enabled to support x2apic and Intr-remapping.
[    0.188070] DMAR-IR: Enabled IRQ remapping in x2apic mode
```

或者执行

```bash
dmesg | grep iommu
```

如果能看到类似内容，也说明 IOMMU 开启成功：

```bash
[    0.000000] Command line: BOOT_IMAGE=/boot/vmlinuz-6.2.16-3-pve root=/dev/mapper/pve-root ro quiet intel_pstate=disable intel_iommu=on iommu=pt pcie_acs_override=downstream pci=realloc=off
[    0.066508] Kernel command line: BOOT_IMAGE=/boot/vmlinuz-6.2.16-3-pve root=/dev/mapper/pve-root ro quiet intel_pstate=disable intel_iommu=on iommu=pt pcie_acs_override=downstream pci=realloc=off
[    0.558404] iommu: Default domain type: Passthrough (set via kernel command line)
[    0.719593] pci 0000:00:02.0: Adding to iommu group 0
[    0.719693] pci 0000:00:00.0: Adding to iommu group 1
[    0.719725] pci 0000:00:01.0: Adding to iommu group 2
[    0.719755] pci 0000:00:01.1: Adding to iommu group 3
......
[    0.720223] pci 0000:05:00.0: Adding to iommu group 17
[    0.720252] pci 0000:06:00.0: Adding to iommu group 18
[    0.720278] pci 0000:08:00.0: Adding to iommu group 19
```

还可以执行命令

```bash
find /sys/kernel/iommu_groups/ -type l 
```

如果能看到很多直通组，说明开启成功：

```bash
/sys/kernel/iommu_groups/17/devices/0000:05:00.0
/sys/kernel/iommu_groups/7/devices/0000:00:1c.0
/sys/kernel/iommu_groups/15/devices/0000:02:00.0
/sys/kernel/iommu_groups/5/devices/0000:00:16.0
......
/sys/kernel/iommu_groups/19/devices/0000:08:00.0
/sys/kernel/iommu_groups/9/devices/0000:00:1c.3
```

