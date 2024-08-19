---
title: "安装黑群晖"
linkTitle: "安装"
weight: 10
date: 2021-01-18
description: >
  在 pve 下安装黑群晖
---

备注： 由于 https://github.com/fbelavenuto/arpl/ 仓库已经归档不再更新，接替它的是新仓库 https://github.com/fbelavenuto/arpl，因此安装方式按照新仓库的要求来。

## 下载

下载地址：

https://github.com/RROrg/rr/releases

pve 下选择 img 格式，下载包是 zip 格式，下载完成后解压缩得到 img 格式文件 rr-24.8.2.img.zip，大概约 480MB，解压缩后得到的 rr.img 文件大概是 3.5GB。上传到 pve 下。 


## 安装

常规方式安装，建立虚拟机，然后删除硬盘，倒入上面上传的 pve img 文件：

```bash
qm importdisk 1003 /var/lib/vz/template/iso/rr.img local
```

然后修改硬盘大小, 增加 128g，这个是系统盘，不用太大。

增加一个新硬盘作为数据盘（如果不增加群晖启动后会因为找不到硬盘而无法配置），大小为 1.5T （1536G），类型为 sata。

启动后选择第一项 configure loader，然后会出现提示，打开页面

http://192.168.2.214:7681/

正常配置，"choose a model"时我选择了最新的，

然后卡在 "Ramdisk打补丁失败，请升级引导版本并重试"。 

## 配置


## 参考资料

- [2024年PVE8最新安装使用指南|安装黑群晖｜img格式镜像安装](https://post.smzdm.com/p/al82z7ee/)
