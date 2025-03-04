---
title: "设置网络代理"
linkTitle: "网络代理"
weight: 55
date: 2023-07-09
description: >
  设置网络代理方便在必要时开启代理
---

安装完成 zsh 之后，顺便把代理配置上，必要时开启。

```bash
vi ~/.zshrc
```

增加以下内容：

```bash
# proxy
alias proxyon='export all_proxy=socks5://192.168.0.1:7891;export http_proxy=http://192.168.0.1:7890;export https_proxy=http://192.168.0.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,.lan,192.168.0.0/16,10.0.0.0/16'
alias proxyoff='unset all_proxy http_proxy https_proxy no_proxy'
```

重新载入：

```bash
source ~/.zshrc
```

