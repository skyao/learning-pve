---
title: "debian12 开发模板"
linkTitle: "开发模板"
weight: 20
date: 2025-03-16
description: >
  debian12 pve 开发模板
---

## 说明

debian pve 的开发模板，基于 debian 12 基础模板，包含软件开发的各种工具和sdk。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| debian12 | dev | 02 | 

## 版本更新

### v01/v02

| 模板编号 | 模板名称 | 模板使用场所 |
| -------- | -------- | -------- |
| 0201 | template-debian12-dev-v01 | 苏州汾湖（192.168.3.1） |
| 0202 | template-debian12-dev-v02 | 广州南沙（192.168.0.1） |

#### 模板说明

name: template-debian12-dev-v01

Development pve template for debian 12.

Installed software:

- sdkman/maven/pip/npm/
- docker/docker-compose/kubectl

Supported languages: Golang/Java/Rust/Python/Nodejs

Build-time: 2025-05-08


