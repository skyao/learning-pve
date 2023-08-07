---
title: "更新CPU微码"
linkTitle: "微码"
weight: 10
date: 2023-07-09
description: >
  更新CPU微码
---

pve 启动时，看到屏幕上有提示要求更新 microcode。

## 添加仓库

需要添加 unstable repo：

```bash
echo "deb http://deb.debian.org/debian/ unstable non-free-firmware" > /etc/apt/sources.list.d/debian-unstable.list
```

执行更新：

```bash
apt update && apt list --upgradable
```

安装微码，intel 选择：

```bash
apt -y install intel-microcode
```

amd 选择：

```bash
apt -y install amd64-microcode
```

更新完之后重启，发现之前报告要求更新 microcode 的信息消失了，搞定。

记得把 unstable 仓库删除，避免不小心更新到这个仓库中的其他软件。

```bash
rm /etc/apt/sources.list.d/debian-unstable.list
```

## 引发问题

安装上面的方式安装微码之后，即使注释或者删除 debian-unstable.list 文件，在 apt update 时也会出现以下提示：

```bash
$ apt update
Hit:1 https://mirrors.ustc.edu.cn/debian bookworm InRelease
Hit:2 https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian bookworm InRelease
Hit:3 https://mirrors.ustc.edu.cn/debian bookworm-updates InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
N: Repository 'Debian bookworm' changed its 'firmware component' value from 'non-free' to 'non-free-firmware'
N: More information about this can be found online in the Release notes at: https://www.debian.org/releases/bookworm/amd64/release-notes/ch-information.html#non-free-split
```

如果不想继续看到这个提示，可以修改文件：

```bash
vi /etc/apt/apt.conf.d/no-bookworm-firmware.conf
```

输入内容：

```properties
APT::Get::Update::SourceListWarnings::NonFreeFirmware "false";
```

再次执行 apt update 命令就不会再出现这个提示了：

```bash
$ apt update
Hit:1 https://mirrors.ustc.edu.cn/debian bookworm InRelease
Hit:2 https://mirrors.ustc.edu.cn/debian bookworm-updates InRelease
Hit:3 https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian bookworm InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
```



## 参考资料

- [How to Install the Latest Microcode on Proxmox VE (Debian stable)](https://cyrusyip.org/en/post/2023/01/31/install-microcode-on-proxmox/)