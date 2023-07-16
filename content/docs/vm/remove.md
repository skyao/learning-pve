---
title: "删除虚拟机"
linkTitle: "删除"
weight: 40
date: 2021-01-18
description: >
  删除虚拟机
---



## 常规方法

标准的做法是在web界面上，将虚拟机停止之后再删除。

## 强制删除

因为某些原因，导致默认的 local-lvm 被删除。此时提交命令

```bash
cd /etc/pve/qemu-server/
rm 100.conf
```

重启，这个虚拟机就消失了。
