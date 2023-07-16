---
title: "修改swap分区参数"
linkTitle: "swap分区"
weight: 10
date: 2021-01-18
description: >
  修改swap分区参数，更多使用物理内存
---

打开文件：

```bash
vi /etc/sysctl.conf
```

在文件最后增加内容：

```bash
vm.swappiness = 1
```

设置内存剩余多少时使用 swap 分区，数字越小代表多使用物理内存。

```bash
sysctl -p
```

让配置生效。