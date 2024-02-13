---
title: "创建pve集群"
linkTitle: "创建集群"
weight: 10
date: 2021-01-18
description: >
  创建pve集群
---

选择一台机器（通常我会选择软路由所在的机器），开始创建 pve 集群。

## 创建集群

登录 web 界面，选择 "data center"，找到 "cluster" ，然后 "create cluster".

创建完成后，点击 "Join Information"，然后 "copy information"。

## 设置vote

修改这台机器的配置：

```bash
vi /etc/pve/corosync.conf
```

修改 nodelist 中这台机器的 quorum_votes 参数，设置的大一些，保证只要这台机器启动就能满足法定人数的需求：

```json
nodelist {
  node {
    name: skyrouter2
    nodeid: 1
    quorum_votes: 3
    ring0_addr: 192.168.20.9
  }
}
```

比如我这个集群只有三台机器，因此我设置软路由所在机器的 quorum_votes 为 3，保证只要软路由这台机器处于开机状态（同样软路由是24小时开机的），集群的法定人数就足以满足要求，其他机器全部关机都不受影响。

备注：这个操作的主要原因是家庭环境下，其他 pve 机器都是按需开启的，不会24小时在线，因此必须处理法定人数的问题。