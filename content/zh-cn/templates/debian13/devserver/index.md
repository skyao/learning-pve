---
title: "debian13 开发服务器模板"
linkTitle: "开发服务器模板"
weight: 30
date: 2025-11-26
description: >
  开发服务器用来作为辅助开发的服务器，提供开发需要用到的各种工具
---

## 模板说明

debian pve 的开发服务器模板，基于 debian 13 基础模板，包含软件开发的各种工具和sdk。

和开发模板不同，开发服务器不是提供开发环境，而是为开发环境提供支持，典型如 maven 本地仓库/docker 本地仓库/数据库/消息队列等。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| debian13 | devserver | 02 | 

### 制作方法

参考 debian 学习笔记的模板制作方法文档：

https://skyao.net/learning-debian/docs/develop/templates/devserver/

## 版本更新

### v01

| 模板编号 | 模板名称 | 
|  -------- | -------- | 
| 990201 | template-debian13-devserver-v01 | 

#### 模板说明

Devserver pve template for debian 13.

Installed software:

- docker/docker-compose/habor
- sdkman/nexus

Build-time: 2025-11-27


