---
title: "为不同端口分别配置VF"
linkTitle: "为不同端口分别配置"
weight: 30
date: 2021-01-18
description: >
  在 pve 下为不同端口分别配置 SR-IOV 的 VF
---



## 背景

之前的 sr-iov 配置，对于有两个 port （即双头网卡）的网卡是以相同方式进行配置的，这样产生的 vf 对应到不同的端口。

```bash
options mlx4_core num_vfs=4,4,0 port_type_array=2,2 probe_vf=4,4,0 probe_vf=4,4,0
```



有些，我们需要对不同的端口进行不一样的配置，比如可能只使用其中的一个端口，或者某一个端口可能就不使用 vf 了。



## 配置方式

参考文章 [HOWTO CONFIGURE SR-IOV VFS ON DIFFERENT CONNECTX-3 PORTS](https://enterprise-support.nvidia.com/s/article/howto-configure-sr-iov-vfs-on-different-connectx-3-ports)

### 为单头网卡配置8个 VF

编辑 `/etc/modprobe.d/mlx4.conf` ，设置

```properties
options mlx4_core port_type_array=2 num_vfs=8 probe_vf=8 log_num_mgm_entry_size=-1
```

### 在双头网卡上配置8个VF, 都在端口1上生效

```properties
options mlx4_core port_type_array=2,2 num_vfs=8,0,0 probe_vf=8,0,0 log_num_mgm_entry_size=-1
```

### 在双头网卡上配置8个VF, 4个VF在端口1上生效，4个VF在端口2上生效

```properties
options mlx4_core port_type_array=2,2 num_vfs=4,4,0 probe_vf=4,4,0 log_num_mgm_entry_size=-1
```

### 在双头网卡上配置8个VF, 2个VF在端口1上生效，2个VF在端口2上生效，还有4个VF在两个端口上都生效

```properties
options mlx4_core port_type_array=2,2 num_vfs=2,2,4 probe_vf=2,2,4 log_num_mgm_entry_size=-1
```





## 参考资料

- [HOWTO CONFIGURE SR-IOV VFS ON DIFFERENT CONNECTX-3 PORTS](https://enterprise-support.nvidia.com/s/article/howto-configure-sr-iov-vfs-on-different-connectx-3-ports)
