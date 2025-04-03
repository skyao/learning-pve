---
title: "debian12 开发服务器模板"
linkTitle: "开发服务器模板"
weight: 30
date: 2025-03-16
description: >
  开发服务器用来作为辅助开发的服务器，提供开发需要用到的各种工具
---

## 说明

debian pve 的开发服务器模板，基于 debian 12 基础模板，包含软件开发的各种工具和sdk。

和开发模板不同，开发服务器不是提供开发环境，而是为开发环境提供支持，典型如 maven 本地仓库/docker 本地仓库/数据库/消息队列等。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| debian12 | devserver | 03 | 

## 版本更新

### v01

| 操作系统 | 模板类型 | 模板类型编号 |  模板编号 | 模板名称 | 
| -------- | -------- | -------- | -------- | -------- | 
| debian12 | devserver | 03 | 0301 | template-debian12-devserver-v01 | 

#### 模板说明

name: template-debian12-devserver-v01

Devserver pve template for debian 12.

Installed software:

- docker/docker-compose/habor
- sdkman/java17/maven/artifactory

Supported languages:

- Java

Build-time: 2025-04-02

### 构建方法

- docker/docker-compose: https://skyao.io/learning-docker/docs/installation/debian12/
- habor: https://skyao.io/learning-docker/docs/repository/habor/
