---
title: "安装配置zsh"
linkTitle: "zsh"
weight: 50
date: 2023-07-09
description: >
  安装zsh/ohmyzsh，并配置proxy信息
---

用习惯 zsh 和 ohmyzsh 了，pve 下希望继续保持相同的体验。

## 安装zsh

和 ubuntu 下类似，但是有些命令会报错，多执行几次就好了。

参考：

https://skyao.io/learning-ubuntu-server/docs/installation/basic/zsh

```bash
apt -y install zsh zsh-doc
```

## 安装ohmyzsh

然后安装ohmyzsh:

```bash
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安全起见，临时设置一下代理：

```bash
export all_proxy=socks5://192.168.0.1:7891;export http_proxy=http://192.168.0.1:7890;export https_proxy=http://192.168.0.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,192.168.0.0/16,10.0.0.0/16
```

## 配置 ohmyzsh

参考：

https://skyao.io/learning-ubuntu-server/docs/installation/basic/zsh

完成后退出登录，然后重新登录进来，发现熟悉的 zsh 回来了。