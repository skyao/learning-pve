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
apt install -y gcc make dkms
apt install -y pve-headers-$(uname -r)
apt install --fix-broken
```





## 工具类

```bash
apt install -y unzip 
```



## 网络类

### iperf

```bash
apt install -y net-tools iperf iperf3
```

Iperf3 安装时会询问是否系统服务（自动启动），选择 yes，这样方便需要时排查网络。

### sftp

pve 默认是关闭 sftp 的，需要手动开启：

```bash
vi /etc/ssh/sshd_config
```

找到 `Subsystem sftp /usr/lib/openssh/sftp-server` 这一行，注释掉，然后新加一行内容：

```bash
# Subsystem sftp /usr/lib/openssh/sftp-server
Subsystem sftp internal-sftp
```

重启 ssh 服务：

```bash
/etc/init.d/ssh restart
```

之后用支持 sftp 的客户端连接即可，比如在 windows 上我现在一般使用 filezilla pro 连接。



