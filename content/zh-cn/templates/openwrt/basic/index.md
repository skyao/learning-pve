---
title: "openwrt 基础模板"
linkTitle: "基础模板"
weight: 10
date: 2026-08-06
description: >
  openwrt pve 基础模板
---


## 说明

openwrt pve 的基础模板，只包含最基本的软件和设置。

bleachwrt-mod 为付费定制的特殊版本，只包含自己需要的插件。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| bleachwrt-mod | basic | 01 | 


## 版本更新

### v01

| 操作系统 | 模板类型 | 模板类型编号 |  模板编号 | 模板名称 | 
| -------- | -------- | -------- | -------- | -------- | 
| bleachwrt-mod | basic | 01 | 0101 | template-bleachwrt-mod-basic-v01 | 

#### 模板说明

name: template-bleachwrt-mod-basic-v01

Basic pve template for openwrt（bleachwrt mod）.

Builtin software:

- tencent ddns
- openclash
- zerotier
- clouddrive2

Config:

- change password of root
- change gateway to 192.168.3.1

Build-time: 2025-08-09

#### 更新说明

- 修改root密码
- 设置网关地址为 192.168.3.1
- 配置 clouddrive2，开启 nfs 和 smb 共享
- 配置 openclash
- zerotier 保持已安装未配置状态

#### 制作方法

参考 openwrt 学习笔记的安装文档 

https://skyao.net/learning-openwrt/docs/installation/pve