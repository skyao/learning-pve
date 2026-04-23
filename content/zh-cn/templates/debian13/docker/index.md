---
title: "debian13 docker模板"
linkTitle: "docker模板"
weight: 40
date: 2025-11-30
description: >
  debian13 pve docker模板
---

## 说明

debian pve 的 docker 模板，基于 debian 13 basic 模板，包含 docker 运行环境。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| debian13 | docker | 03 | 

## 版本更新

### v02

| 模板编号 | 模板名称 |
| -------- | -------- | 
| 990302 | template-debian13-docker-v02 | 

#### 模板说明

Docker pve template for debian 13.

Udpate software version:

- docker： v29.4.1-1
- docker-compose： v5.1.3
- cri-dockerd： 0.4.2

Build-time: 2026-04-22

### v01

| 模板编号 | 模板名称 |
| -------- | -------- | 
| 990301 | template-debian13-docker-v01 | 

#### 模板说明

Docker pve template for debian 13.

Installed software:

- docker： v29.1.1-1
- docker-compose： v2.40.3
- cri-dockerd： 0.4.0.3-0

Configuration:

- use harbor as image registry

Build-time: 2025-11-30


