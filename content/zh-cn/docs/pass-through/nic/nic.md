---
title: "设置网卡直通并检查是否生效"
linkTitle: "设置和检查"
weight: 10
date: 2021-01-18
description: >
  设置pve下的网卡直通并检查是否生效
---



## 直通网卡

### 创建虚拟机

特别注意，创建虚拟机时要选择 q35

![](images/q35.png)

在虚拟机的 hardware 设置中，添加 pci device：

![](images/add-pci-device.png)

注意勾选 All functions 以直通网卡上的所有网口，另外对于 pcie 设置，要勾选 “PCI-Express”。

直通两块HP544+网卡之后，虚拟机的设置如下：

![](images/overview.png)



