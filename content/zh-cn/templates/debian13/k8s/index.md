---
title: "debian13 k8s 模板"
linkTitle: "k8s 模板"
weight: 50
date: 2025-11-27
description: >
  debian13 pve k8s 模板
---

## 模板说明

debian pve 的 k8s 模板，基于 debian 13 docker 模板，提供 k8s 的预热安装环境。

| 操作系统 | 模板类型 | 模板类型编号 |  
| -------- | -------- | -------- | 
| debian13 | k8s | 04 | 

### 制作方法

参考 debian 学习笔记的模板制作方法文档：

https://skyao.net/learning-debian/docs/develop/templates/k8s/

## 版本更新

### v02

| 模板编号 | 模板名称 | 
| -------- | -------- | 
| 990402 | template-debian13-k8s-v02 | 

#### 模板说明

name: template-debian13-k8s-v02

PVE template for K8s prewarm on debian 13.

Update software version:

- helm: v3.20.2
- kubeadm / kubelet / kubectl: v1.35

Update prewarm files for:

- k8s: v1.35
- kubernetes-dashboard: 已被废弃不再安装
- metrics-server: v0.8.1

Build-time: 2026-04-22

### v01

| 模板编号 | 模板名称 | 
| -------- | -------- | 
| 990401 | template-debian13-k8s-v01 | 

#### 模板说明

name: template-debian13-k8s-v01

PVE template for K8s prewarm on debian 13.

Installed software:

- helm: v3.19.2
- kubeadm / kubelet / kubectl: v1.34.2

Installation files and prewarm files for:

- k8s: v1.34.2
- kubernetes-dashboard: 7.14.0
- metrics-server: v0.8.0

Build-time: 2025-11-30


