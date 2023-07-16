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

https://skyao.io/learning-ubuntu-server/docs/installation/basic/zsh.html

```bash
apt install zsh zsh-doc
```

## 安装ohmyzsh

然后安装ohmyzsh:

```bash
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

比较奇怪会报错，重试几次又成功了：

```bash
root@skyserver2:~# sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
--2023-07-12 17:41:20--  https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 199.232.68.133, 185.199.111.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|199.232.68.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 18209 (18K) [text/plain]
Saving to: 'STDOUT'

-                                           100%[========================================================================================>]  17.78K  --.-KB/s    in 0.04s   

2023-07-12 17:41:22 (400 KB/s) - written to stdout [18209/18209]

Cloning Oh My Zsh...
fatal: unable to access 'https://github.com/ohmyzsh/ohmyzsh.git/': Recv failure: Connection reset by peer
/root
Error: git clone of oh-my-zsh repo failed
root@skyserver2:~# sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
--2023-07-12 17:41:33--  https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 199.232.68.133, 185.199.111.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|199.232.68.133|:443... connected.
GnuTLS: Error in the pull function.
Unable to establish SSL connection.
root@skyserver2:~# sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
--2023-07-12 17:41:36--  https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 199.232.68.133, 185.199.111.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|199.232.68.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 18209 (18K) [text/plain]
Saving to: 'STDOUT'

-                                           100%[========================================================================================>]  17.78K  --.-KB/s    in 0.05s   

2023-07-12 17:41:38 (350 KB/s) - written to stdout [18209/18209]

Cloning Oh My Zsh...
remote: Enumerating objects: 1340, done.
remote: Counting objects: 100% (1340/1340), done.
remote: Compressing objects: 100% (1289/1289), done.
remote: Total 1340 (delta 30), reused 1135 (delta 27), pack-reused 0
Receiving objects: 100% (1340/1340), 1.99 MiB | 1.41 MiB/s, done.
Resolving deltas: 100% (30/30), done.
From https://github.com/ohmyzsh/ohmyzsh
 * [new branch]      master     -> origin/master
branch 'master' set up to track 'origin/master'.
Already on 'master'
/root

Looking for an existing zsh config...
Using the Oh My Zsh template file and adding it to /root/.zshrc.

Time to change your default shell to zsh:
Do you want to change your default shell to zsh? [Y/n] Y
Changing your shell to /usr/bin/zsh...
Shell successfully changed to '/usr/bin/zsh'.
```

安全起见，临时设置一下代理：

```bash
export all_proxy=socks5://192.168.0.1:7891;export http_proxy=http://192.168.0.1:7890;export https_proxy=http://192.168.0.1:7890;export no_proxy=127.0.0.1,localhost,local,.local,192.168.0.0/16,10.0.0.0/16
```

## 配置 ohmyzsh

参考：

https://skyao.io/learning-ubuntu-server/docs/installation/basic/zsh.html

只是注意命令可能会报错（也许是我今天这边的网络表现不稳定？），多重复几次即可。

完成后退出登录，然后重新登录进来，发现熟悉的 zsh 回来了。