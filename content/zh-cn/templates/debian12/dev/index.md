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

Dev pve template for debian 12.

Installed software:

- xxx

Supported languages:

- Golang
- Java
- Rust
- Python
- Nodejs

Install development tools:

- linux headers

- apt install zlib1g-dev 

System Config:

- add proxyon/proxyoff alias for different locations
- fix path for user root and sky

Build-time: 2025-03-16

#### 构建方法

安装 linux headers:

```bash
sudo apt-get install linux-headers-$(uname -r)
```

安装各种开发包：

```bash
sudo apt install zlib1g-dev
```
