---
title: "debian12 k8s 模板"
linkTitle: "k8s 模板"
weight: 40
date: 2025-05-12
description: >
  debian12 pve k8s 模板
---

## 模板说明

debian pve 的 k8s 模板，基于 debian 12 基础模板，提供 k8s 的预热安装环境。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| debian12 | k8s | 04 | 

### 制作方法

参考 debian 学习笔记的模板制作方法文档：

https://skyao.io/learning-debian/templates/basic/

## 版本更新

### v01/v02

| 模板编号 | 模板名称 | 模板使用场所 |
| -------- | -------- | -------- |
| 0401 | template-debian12-k8s-v01 | 苏州汾湖（192.168.3.1） |
| 0402 | template-debian12-k8s-v02 | 广州南沙（192.168.0.1） |

#### 模板说明

name: template-debian12-k8s-v01

PVE template for K8s prewarm on debian 12.

Installed software:

- docker/docker-compose/cri-dockerd/helm
- kubeadm / kubelet / kubectl: v1.33

Installation files and prewarm files for:

- k8s: v1.33
- kubernetes-dashboard: 7.12.0
- metrics-server: v0.7.2

Build-time: 2025-05-12


