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