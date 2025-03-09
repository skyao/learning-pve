---
title: "openwrt 基础模板"
linkTitle: "基础模板"
weight: 10
date: 2025-03-04
description: >
  openwrt pve 基础模板
---


## 说明

openwrt pve 的基础模板，只包含最基本的软件和设置。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| openwrt | basic | 01 | 


## 版本更新

### v01

| 操作系统 | 模板类型 | 模板类型编号 |  模板编号 | 模板名称 | 
| -------- | -------- | -------- | -------- | -------- | 
| openwrt | basic | 01 | 0101 | template-openwrt-basic-v01 | 

#### 模板说明

name: template-openwrt-basic-v01

Basic pve template for openwrt.

Installed software:

- tencent ddns

Config:

- change password of root
- change gateway to 192.168.3.1

Build-time: 2025-03-09

#### 更新说明

- 安装腾讯ddns
- 修改root密码
- 设置网关地址为 192.168.3.1

#### 制作方法

参考 openwrt 学习笔记的安装文档 

https://skyao.io/learning-openwrt/docs/installation/pve