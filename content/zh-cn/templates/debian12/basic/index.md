---
title: "debian12 基础模板"
linkTitle: "基础模板"
weight: 10
date: 2025-05-07
description: >
  debian12 pve 基础模板
---

## 模板说明

debian pve 的基础模板，只包含最基本的软件和设置。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| debian12 | basic | 01 | 

### 制作方法

参考 debian12 学习笔记的模板制作方法文档：

https://skyao.io/learning-debian/templates/basic/ 

## 版本更新

### v03/v04

| 模板编号 | 模板名称 | 模板使用场所 |
| -------- | -------- | -------- |
| 0103 | template-debian12-basic-v03 | 苏州汾湖（192.168.3.1） |
| 0104 | template-debian12-basic-v04 | 广州南沙（192.168.0.1） |

#### 模板说明-v03

name: template-debian12-basic-v03

Basic pve template for debian 12.

Upgraded to debian12.10 and kernel 6.1.0-34 on 2025-05-07

Installed software:

- timeshift
- zsh/ohmyzsh
- git/unzip/zip/curl/iperf/iperf3/dkms
- sftp-server/nfs server/nfs client/samba server/

Config:

- add proxyon/proxyoff alias for different locations
- fix path for user root and sky

Build-time: 2025-05-07

#### 模板说明-v04

v03 迁移到其他区域时，需要进行部分修改，主要是代理设置。

修改内容如下：

- `/home/sky/.zshrc` 文件中，proxyon 设置为 proxyon-nansha
- `/root/.zshrc` 文件中，proxyon 设置为 proxyon-nansha

修改之后，重新制作模板并命名为 template-debian12-basic-v04。

