---
title: "debian12 基础模板"
linkTitle: "基础模板"
weight: 10
date: 2025-03-04
description: >
  debian12 pve 基础模板
---


## 说明

debian pve 的基础模板，只包含最基本的软件和设置。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| debian12 | basic | 01 | 


## 版本更新

### v01

| 操作系统 | 模板类型 | 模板类型编号 |  模板编号 | 模板名称 | 
| -------- | -------- | -------- | -------- | -------- | 
| debian12 | basic | 01 | 0101 | template-debian12-basic-v01 | 

#### 模板说明

name: template-debian12-basic-v01

Basic pve template for debian 12.

Upgraded to debian12.9 and apt updated to latest on 2025-03-09

Installed software:

- timeshift
- zsh/ohmyzsh
- git
- dkms
- iperf/iperf3
- unzip zip curl

Config:

- add proxyon/proxyoff alias for different locations
- fix path for user root and sky

Build-time: 2025-03-09

#### 更新说明

- 升级到 debian12.9
- 更新 apt 到最新
- 安装 timeshift, zsh/ohmyzsh, git, dkms, iperf/iperf3, unzip zip curl
- 添加 proxyon/proxyoff alias for different locations
- 修复 path 不足问题，包括用户 root 和 sky

#### 制作方法

参考 debian12 学习笔记的安装文档 

https://skyao.io/learning-pve/docs/installation/ 

迁移到其他区域时，需要进行部分修改，主要是代理设置。

修改内容如下：

- `/home/sky/.zshrc` 文件中，proxyon 的设置
- `/root/.zshrc` 文件中，proxyon 设置

