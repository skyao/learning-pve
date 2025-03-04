---
title: "基础模板"
linkTitle: "基础模板"
weight: 10
date: 2024-11-14
description: >
  debian pve 基础模板
---


## 说明

debian pve 的基础模板，只包含最基本的软件和设置。

## 版本更新

### v02

name: template-debian12-basic-v02

Upgraded to debian12.9 and apt updated to latest on 2025-03-04

Installed software:

- timeshift
- zsh/ohmyzsh
- git
- dkms
- iperf/iperf3
- unzip zip curl

Config:

- add proxyon/proxyoff alias for different locations

Build-time: 2025-03-04

### v01

name: template-debian12-basic-v01

pve templete for debian 12 (v01)

Installed as debian12.8, and apt updated to latest on 2024-11-14

Installed software:

- timeshift
- zsh/ohmyzsh
- git
- dkms
- iperf/iperf3
- unzip zip curl

Build-time: 2024-11-24

制作方式：参考本学习笔记的安装文档