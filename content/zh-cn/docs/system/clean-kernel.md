---
title: "清理不再使用的内核"
linkTitle: "清理内核"
weight: 35
date: 2021-01-18
description: >
  清理不再使用的内核
---



## 脚本操作

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

## 手工操作



参考： https://asokolsky.github.io/proxmox/kernels.html



