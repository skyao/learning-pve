---
title: "关闭虚拟机"
linkTitle: "关闭"
weight: 60
date: 2021-01-18
description: >
  关闭虚拟机
---


### 强行关闭虚拟机

遇到情况是，在虚拟机操作系统安装界面，因为有配置错误需要关闭虚拟机再修改虚拟机配置，结果发现在操作系统安装界面会无法关闭虚拟机。

强行关闭虚拟机的方式：

- 登录节点：可以从 web 或者 ssh
- 执行命令： `qm stop 100` （100 是虚拟机编号，自行替代）



### 修改虚拟机配置文件

重启 pve，在启动菜单中选择高级选项，然后选择恢复模式。

恢复模式启动后，输入 root 账号密码登录。

启动 pve-cluster ：

```bash
service pve-cluster start
```

进入 qemu-server：

```bash
cd /etc/pve/qemu-server
```

这时就可以直接编辑和修改虚拟机配置文件了。