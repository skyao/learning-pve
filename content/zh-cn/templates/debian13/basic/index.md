---
title: "debian13 基础模板"
linkTitle: "基础模板"
weight: 10
date: 2025-05-07
description: >
  debian13 pve 基础模板
---

## 模板说明

debian pve 的基础模板，只包含最基本的软件和设置。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| debian13 | basic | 00 | 

### 制作方法

参考 debian 学习笔记的模板制作方法文档：

https://skyao.net/learning-debian/docs/develop/templates/basic/

## 版本更新

#### 模板说明-v01

| 模板编号 | 模板名称 |
| -------- | -------- | 
| 990001 | template-debian13-basic-v01 | 

Basic pve template for debian 13.

Upgraded to debian13.2 and kernel 6.12.48-1 on 2025-11-26

Installed software:

- timeshift
- zsh/ohmyzsh
- git/unzip/zip/curl/iperf/iperf3/dkms
- sftp-server/nfs server/nfs client/samba server/

Config:

- add proxyon/proxyoff alias for different locations
- fix path for user root and sky

Build-time: 2025-11-27



