---
title: "debian12 开发服务器模板"
linkTitle: "开发服务器模板"
weight: 30
date: 2025-03-16
description: >
  开发服务器用来作为辅助开发的服务器，提供开发需要用到的各种工具
---

## 模板说明

debian pve 的开发服务器模板，基于 debian 12 基础模板，包含软件开发的各种工具和sdk。

和开发模板不同，开发服务器不是提供开发环境，而是为开发环境提供支持，典型如 maven 本地仓库/docker 本地仓库/数据库/消息队列等。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| debian12 | devserver | 03 | 

### 制作方法

参考 debian12 学习笔记的模板制作方法文档：

https://skyao.net/learning-debian/templates/devserver/ 

## 版本更新

### v01/v02

| 模板编号 | 模板名称 | 模板使用场所 |
|  -------- | -------- | -------- |
| 0301 | template-debian12-devserver-v01 | 苏州汾湖（192.168.3.1） |
| 0302 | template-debian12-devserver-v02 | 广州南沙（192.168.0.1） |

#### 模板说明

name: template-debian12-devserver-v01

Devserver pve template for debian 12.

Installed software:

- docker/docker-compose/habor
- sdkman/nexus

Installed SDK for languages: Java/Golang/Rust/Python/Nodejs

Build-time: 2025-05-07


