---
title: "安装黑群晖"
linkTitle: "[归档]安装"
weight: 100
date: 2021-01-18
description: >
  在 pve 下安装黑群晖
---

备注： 由于 https://github.com/fbelavenuto/arpl/ 仓库已经归档不再更新，因此这个安装方式也只能归档

## 下载

下载地址：

https://github.com/fbelavenuto/arpl/releases

pve 下选择 img 格式，下载包是 zip 格式，下载完成后解压缩得到 img 格式文件 arpl.img，大概约 1 GB。上传到 pve 下。

## 安装

常规方式安装，建立虚拟机，然后删除硬盘，倒入上面上传的 pve img 文件：

```bash
qm importdisk 103 /var/lib/vz/template/iso/arpl.img local
```

然后修改硬盘大小为 128g，这个是系统盘，不用太大。

增加一个新硬盘作为数据盘（如果不增加群晖启动后会因为找不到硬盘而无法配置），大小为2T。

启动后打开页面

http://192.168.20.160:7681/

正常配置，我选择了最新的，安装完成后打开

http://192.168.20.160

继续配置。

## 配置


## 参考资料

- [从零开始的all in one之pve安装黑群晖](https://zhuanlan.zhihu.com/p/639066104)
