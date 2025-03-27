---
title: "windows10 nas 模板"
linkTitle: "nas模板"
weight: 20
date: 2025-03-25
description: >
  windows10 pve nas 模板
---

## 说明

windows10 pve 的 nas 模板，用于 nas 服务器。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| windows10 | nas | 22 | 

## 版本更新

### v01

| 操作系统 | 模板类型 | 模板类型编号 |  模板编号 | 模板名称 | 
| -------- | -------- | -------- | -------- | -------- | 
| windows10 | nas | 22 | 2201 | template-windows10-nas-v01 | 

name: template-windows10-nas-v01

pve templete for windows 10 nas server

Based on template-windows10-basic-v01, and updated to latest on 2025-03-25

Build-time: 2025-03-25

Installed software:

- hanewin nfs 服务器/nginx服务器
- plex media server/完美解码
- zerotier/qBittorrent/百度云盘 
- scarletbook/dff2dsf/audio-converter/Tag&Rename/tinymediamanager

## 模板实例

物理机 nas 机器有：

- skynas： 广州南沙，两块16t硬盘（资料和影视各一），一块3t硬盘（pt下载）
- skynas2：广州天河，两块18t硬盘（资料），三块16t硬盘（影视）

其他为 pve 下的虚拟机，使用 template-windows10-nas 模板。

### skynas3-tianhe

天河软路由机器上的 nas，用来存放学习资料，音乐，这是唯一一台24小时开机的机器。

磁盘为一块 2t 的 sata ssd 硬盘。

### skynas4-fenhu

用在汾湖 switch98 机器上，用来充当汾湖住所的多媒体 nas 服务器和工作用 nas 服务器。

磁盘为一块 1t 的 nvme ssd 硬盘（三星pm983）和一块 16t 机械硬盘。

### skynas5-nansha

用在南沙 switch99 机器上，用来充当南沙住所的工作用 nas 服务器。

磁盘为一块 1t 的 nvme ssd 硬盘（三星pm983）和一块 16t 机械硬盘。

