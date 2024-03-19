---
title: "安装常用软件"
linkTitle: "常用软件"
weight: 65
date: 2023-07-09
description: >
  PVE安装完成后继续安装常用的各种软件
---



## 系统类

安装 dkms 和用于 pve 的 linux-headers：

```bash
apt install -y dkms
apt install -y pve-headers-$(uname -r)
apt install --fix-broken
```





## 工具类

```bash
apt install -y unzip 
```



## 网络类



```bash
apt install -y net-tools iperf iperf3
```





## 开发类

