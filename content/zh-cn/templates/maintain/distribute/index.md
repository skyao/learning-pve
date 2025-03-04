---
title: "模板分发"
linkTitle: "分发"
weight: 60
date: 2025-03-04
description: >
  debian pve 模板的分发
---

### pve 到 pve 复制

直接复制 pve 机器上的备份文件到其他 pve 机器：

```bash
scp /var/lib/vz/dump/vzdump-qemu-10102-2025_03_04-11_26_15.* root@192.168.3.129:/var/lib/vz/dump/
```

scp /var/lib/vz/dump/vzdump-qemu-10102-2025_03_04-11_25_58.* root@192.168.0.19:/var/lib/vz/dump/