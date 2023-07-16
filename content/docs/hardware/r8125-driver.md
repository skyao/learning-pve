---
title: "安装r8125驱动"
linkTitle: "r8125驱动"
weight: 20
date: 2023-07-09
description: >
  为relteck 8125 2.5G网卡安装驱动
---



## 准备工作



```bash
apt install -y dkms
apt install -y pve-headers-$(uname -r)
apt install --fix-broken
```



## 下载驱动

https://github.com/awesometic/realtek-r8125-dkms/releases

```bash
mkdir ~/temp
cd ~/temp
wget https://github.com/awesometic/realtek-r8125-dkms/releases/download/9.010.01-2/realtek-r8125-dkms_9.010.01-2_amd64.deb
```

## ~~通过安装包安装驱动~~

```bash
dpkg -i realtek-r8125-dkms_9.010.01-2_amd64.deb
```

输出为：

```bash
dpkg -i realtek-r8125-dkms_9.010.01-2_amd64.deb
Selecting previously unselected package realtek-r8125-dkms.
(Reading database ... 86910 files and directories currently installed.)
Preparing to unpack realtek-r8125-dkms_9.010.01-2_amd64.deb ...
Unpacking realtek-r8125-dkms (9.010.01-2) ...
Setting up realtek-r8125-dkms (9.010.01-2) ...
locale: Cannot set LC_ALL to default locale: No such file or directory
Loading new realtek-r8125-9.010.01 DKMS files...
Deprecated feature: REMAKE_INITRD (/usr/src/realtek-r8125-9.010.01/dkms.conf)
Building for 6.2.16-4-pve
Building for architecture amd64
Building initial module for 6.2.16-4-pve
Deprecated feature: REMAKE_INITRD (/var/lib/dkms/realtek-r8125/9.010.01/source/dkms.conf)
Done.
Deprecated feature: REMAKE_INITRD (/var/lib/dkms/realtek-r8125/9.010.01/source/dkms.conf)
Deprecated feature: REMAKE_INITRD (/var/lib/dkms/realtek-r8125/9.010.01/source/dkms.conf)

r8125.ko:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/6.2.16-4-pve/updates/dkms/
depmod...
```

这个版本是 9.010。

## 通过源码安装驱动

发现代码里面有更新到 9.011，只是没有打包发布为 deb 格式。但是可以从源码开始安装。

```bash
git clone https://github.com/awesometic/realtek-r8125-dkms.git 
cd realtek-r8125-dkms
./dkms-install.sh
```

输出为：

```bash
./dkms-install.sh 
About to run dkms install steps...
Deprecated feature: REMAKE_INITRD (/usr/src/r8125-9.011.01/dkms.conf)
Creating symlink /var/lib/dkms/r8125/9.011.01/source -> /usr/src/r8125-9.011.01
Sign command: /lib/modules/6.2.16-4-pve/build/scripts/sign-file
Signing key: /var/lib/dkms/mok.key
Public certificate (MOK): /var/lib/dkms/mok.pub
Deprecated feature: REMAKE_INITRD (/var/lib/dkms/r8125/9.011.01/source/dkms.conf)

Building module:
Cleaning build area...
'make' -j16 KVER=6.2.16-4-pve BSRC=/lib/modules/6.2.16-4-pve modules......
Signing module /var/lib/dkms/r8125/9.011.01/build/src/r8125.ko
Cleaning build area...
Deprecated feature: REMAKE_INITRD (/var/lib/dkms/r8125/9.011.01/source/dkms.conf)

r8125.ko:
Running module version sanity check.
 - Original module
 - Installation
   - Installing to /lib/modules/6.2.16-4-pve/updates/dkms/
depmod...
Finished running dkms install steps.
```

## 更换驱动

屏蔽 r8169 驱动：

```bash
sudo sh -c 'echo blacklist r8169 >> /etc/modprobe.d/blacklist_r8169.conf'
```

重启机器，检验：

```bash
lsmod | grep r81  
r8125                 262144  0
r8169                 114688  0
```

对比没有更新驱动的输出：

```bash
lsmod | grep r81  
r8169                 114688  0
```



## 检验

更新之前，客户端和服务器端都使用默认的 r8169 驱动，iperf3 测试速度为 2.35-2.36G。

更新客户端驱动为  r8125 驱动, iperf3 测试速度为 2.35-2.36 G，几乎没有变化。

继续更新服务器端驱动为  r8125 驱动, iperf3 测试速度为 2.35/2.36G，几乎没有变化。

总结：更新驱动未能带来速度方面的直接收益。



## 参考资料

- https://zhuanlan.zhihu.com/p/577161166
- https://www.right.com.cn/forum/forum.php?mod=viewthread&tid=7446026
- https://github.com/awesometic/realtek-r8125-dkms： 鸣谢作者！

