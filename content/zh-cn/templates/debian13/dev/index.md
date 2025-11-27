---
title: "debian13 开发模板"
linkTitle: "开发模板"
weight: 20
date: 2025-03-16
description: >
  debian12 pve 开发模板
---

## 说明

debian pve 的开发模板，基于 debian 13 基础模板，包含软件开发的各种工具和sdk。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| debian13 | dev | 01 | 

## 版本更新

### v01

| 模板编号 | 模板名称 |
| -------- | -------- | 
| 990101 | template-debian13-dev-v01 | 

#### 模板说明

Development pve template for debian 13.

Installed software:

- sdkman/maven/pip/npm/
- docker/docker-compose/kubectl

Supported languages: 

- Golang: go1.25.4
- Java: 25.0.1-zulu / 21.0.9-zulu / 17.0.17-zulu / 11.0.29-zulu / 8.0.472-zulu, maven 3.9.11
- Rust: 1.91.1
- Python: 3.10.19 / 3.11.14 / 3.12.12
- Nodejs: node v24.11.0 / npm 11.6.1

Build-time: 2025-11-26


