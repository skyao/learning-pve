---
title: "删除集群节点"
linkTitle: "删除节点"
weight: 20
date: 2021-01-18
description: >
  将节点从集群中删除
---


参考：

https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_remove_a_cluster_node

注意：

删除节点后，其 SSH 指纹仍将驻留在其他节点的known_hosts中。如果您在重新加入具有相同 IP 或主机名的节点后收到 SSH 错误，请在重新添加的节点上运行一次 pvecm 更新证书，以更新其指纹集群范围。



### 退出节点

登录集群中待退出之外的其他任意一个节点，执行

```bash
pvecm nodes
```

输出如下：

```bash
    Nodeid      Votes Name
         1          1 skyserver (local)
         2          1 skyaio2
         3          1 skyserver2
         4          1 skyserver3
         5          1 skyserver4
         6          1 skyserver5
         7          1 skyserver6
         8          1 skyaio
```

现在来让 skyaio 节点退出。

先关闭 skyaio 节点，确保该节点已经关机。

```bash
pvecm delnode skyaio
```

节点删除之后检验一下：

```bash
pvecm nodes
```

可以看到 skyaio 节点已经不在了：

```bash
    Nodeid      Votes Name
         1          1 skyserver (local)
         2          1 skyaio2
         3          1 skyserver2
         4          1 skyserver3
         5          1 skyserver4
         6          1 skyserver5
         7          1 skyserver6
```

用命令查看集群状态：

```bash
pvecm status
```

可以看到 skyaio 节点已经不在了：

```bash
......
Membership information
----------------------
    Nodeid      Votes Name
0x00000001          1 192.168.0.18 (local)
0x00000002          1 192.168.0.82
0x00000003          1 192.168.0.28
0x00000004          1 192.168.0.38
0x00000005          1 192.168.0.48
0x00000006          1 192.168.0.58
0x00000007          1 192.168.0.68
```

### 清理残存信息

#### 从web页面列表中清除

打开 pve 的 web 页面时，会发现 skyaio 节点还在列表中，但已经无法连接。

依然是登录任意一个节点，

```bash
cd /etc/pve/nodes
ls
skyaio  skyaio2  skyserver  skyserver2  skyserver3  skyserver4  skyserver5  skyserver6
```

删除 skyaio 目录：

```bash
rm -rf skyaio
```

刷新页面即可看到 skyaio 节点消失了。

#### 清除 authorized_keys 和 known_hosts

```bash
cd /etc/pve/priv
```

清理 authorized_keys 

```bash
vi authorized_keys
```

打开后搜索 skyaio，然后删除该行。

```bash
vi known_hosts
```

同样在打开后搜索 skyaio，然后删除该行。

#### 清除其他信息

查找其他位置可能存在的节点信息：

```bash
grep skyaio /etc -r  
```

排除 /etc/pve/.clusterlog 之外，比如这个文件：

```bash
/etc/pve/storage.cfg:	nodes skyserver2,skyaio,skyserver6,skyserver5,skyserver4,skyaio2,skyserver3,skyserver
```

打开后删除 skyaio 即可。