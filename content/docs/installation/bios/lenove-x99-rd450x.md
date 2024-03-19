---
title: "联想rd450x x99双路主板"
linkTitle: "联想x99"
weight: 10
date: 2021-01-18
description: >
  联想rd450x x99双路主板安装PVE的BIOS设置
---



## 背景

在 联想rd450x x99双路主板上安装 pve，有两种情况：

1. 普通机器：插一块pcie转nvme的转换卡接ssd硬盘，插一块网卡（hp544+）
2. AI机器：除了ssd硬盘和网卡外，还要插两块p102显卡

## BIOS设置

先重置 bios，然后开始设置。

### 普通机器设置



### AI 机器设置

#### Advanced 选项

"Advanced" -> "Processor Configuration"

- "Intel Virtualization Technology": enable （默认就是enable），虚拟化支持必须开启。
- X2APIC：enable （默认是 disable），PCIe 硬件直通需要

"Advanced" -> "Advanced Power Management Configuration"

- CPU C statue: "disable"
- Enhanced Halt Statue(C1E): disable
- Package C State Limit: "C0/C1 state"
- CPU C3 report: disable
- CPU C6 report: disable

"Advanced" -> "Memory Configuration"

- Numa: enabled, 多路 CPU 建议开启，提高多路 CPU 运行效率，合理分配负载

"Advanced" -> "Chipset Configuration"

- Intel VT for Directed I/O (VT-D): enable，PCIeE硬件直通必须

"Advanced" -> "PCI/PCIE Configuration"

- Above 4G Decoding: enabled, vGPU 方案需要开启这个选项
- SR-IOV Support: enabled, 网卡虚拟化需要开启这个选项
- PCI-E ASPM Support(PCH): L1 only，让OS管理PCI设备电源，切断宿主BIOS和PCI设备间的管理关系
- PCI-E ASPM Support(IIO): L1 only，同上

"Advanced" -> "Serrial Settings"

- Serial Port 0: disabled，禁用串口
- Serial Port 1: disabled

"Advanced" -> "Wakeup Settings"

- Restore AC Power Loss: Power On，断电后重新通电就自动加电启动机器

"Advanced" -> "Network Stack Configuration"

- Network Stack: Disabled，不需要网络启动功能
- Onboard Lan: disabled，不需要板载网卡

#### Boot 选项

"Boot"

- Setup Prompt Timeout: 5
- Quiet Boot: disabled

"Boot" -> "CSM Configuration"

- CSM support: enabled
- Boot option filter: UEFI only
- Launch PXE OpROM: Do not launch
- Storage: UEFI
- Video: Legacy
- Other PCI devices: UEFI



