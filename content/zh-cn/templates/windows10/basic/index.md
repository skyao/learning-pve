---
title: "windows10 基础模板"
linkTitle: "基础模板"
weight: 10
date: 2025-03-04
description: >
  windows10 pve 基础模板
---

## 说明

windows10 pve 的基础模板，只包含最基本的软件和设置。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| windows10 | basic | 21 | 

## 版本更新

### v02（计划中）

| 操作系统 | 模板类型 | 模板类型编号 |  模板编号 | 模板名称 | 
| -------- | -------- | -------- | -------- | -------- | 
| windows10 | basic | 21 | 2101 | template-windows10-basic-v02 | 

name: template-windows10-basic-v02

pve templete for windows 10 (v02)

Build-time: 待定

Installed software:

- everything/acdsee

System config：

- 解决 non unicode 中文乱码问题： https://skyao.net/learning-windows11/docs/installation/system-settings/#%E4%B8%AD%E6%96%87%E4%B9%B1%E7%A0%81
- windows update 到最新

### v01

| 操作系统 | 模板类型 | 模板类型编号 |  模板编号 | 模板名称 | 
| -------- | -------- | -------- | -------- | -------- | 
| windows10 | basic | 21 | 2101 | template-windows10-basic-v01 | 

name: template-windows10-basic-v01

pve templete for windows 10 (v01)

Installed as windows 10 iot, and updated to latest on 2025-03-04

Build-time: 2025-03-15

Installed drivers:

- pve virtio drivers / virtio windows guest tools

Installed software:

- ssh server/git/zsh/ohmyzsh
- wget/FileZilla Pro/FileZilla Server
- iperf/iperf3
- 7zip/freefilesync


System config：

- 开启远程桌面
- 开启文件共享，共享目录为 download 和 data/shared
- 安装字体：思源黑体，文泉驿正黑，方正兰亭黑，文泉驿微米黑




