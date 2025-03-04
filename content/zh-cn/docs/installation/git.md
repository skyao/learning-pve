---
title: "安装配置git"
linkTitle: "git"
weight: 40
date: 2023-07-09
description: >
  安装git并设置代理
---



## 安装 git

```bash
apt -y install git 
```

## 配置代理

```bash
vi ~/.ssh/config
```

增加以下内容：

```properties
Host github.com
HostName github.com
User git
# http proxy
#ProxyCommand socat - PROXY:192.168.0.1:%h:%p,proxyport=7890
# socks5 proxy
ProxyCommand nc -v -x 192.168.0.1:7891 %h %p
```

