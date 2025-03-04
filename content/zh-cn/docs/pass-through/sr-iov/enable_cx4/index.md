---
title: "配置 CX-4 网卡以开启 SR-IOV"
linkTitle: "开启 SR-IOV: cx-4"
weight: 25
date: 2021-01-18
description: >
  配置 Mellanox connextx-4 网卡配置以便在 pve 中开启网卡的 SR-IOV 功能
---



## 准备工作

### 更新网卡固件

对于 mcx4121 网卡，参考：

https://skyao.io/learning-computer-hardware/nic/cx4121a/firmware/

### 设置主板bios

需要在主板中设置开启以下功能：

- 虚拟机支持
- vt-d
- SR-IOV

### 安装 pve-headers

对于 PVE，还需要安装额外的 pve-headers，否则后续安装 mft 时会报错，提示要安装 linux-headers 和 linux-headers-generic：

```bash
./install.sh
-E- There are missing packages that are required for installation of MFT.
-I- You can install missing packages using: apt-get install linux-headers linux-headers-generic
```

安装：

```bash
apt install -y gcc make dkms
apt install -y pve-headers-$(uname -r)
apt install --fix-broken
```

安装完成之后，需要重启，否则直接安装 mft，依然会继续同样报错。

### 安装 mft 

对于 cx4 / cx5 等新一点的网卡，可以安装 mft 最新版本。

```bash
mkdir -p ~/work/soft/mellanox
cd ~/work/soft/mellanox

wget --no-check-certificate https://www.mellanox.com/downloads/MFT/mft-4.27.0-83-x86_64-deb.tgz
tar xvf mft-4.27.0-83-x86_64-deb.tgz
cd mft-4.27.0-83-x86_64-deb
```

执行安装脚本:

```bash
./install.sh
```

输出为：

```bash
-I- Removing mft external packages installed on the machine
-I- Removing the packages:  kernel-mft-dkms...
-I- Installing package: /root/temp/mft-4.27.0-83-x86_64-deb/SDEBS/kernel-mft-dkms_4.27.0-83_all.deb
-I- Installing package: /root/temp/mft-4.27.0-83-x86_64-deb/DEBS/mft_4.27.0-83_amd64.deb
-I- Installing package: /root/temp/mft-4.27.0-83-x86_64-deb/DEBS/mft-autocomplete_4.27.0-83_amd64.deb
```

### 下载 mlxup

```bash
cd ~/work/soft/mellanox

wget https://www.mellanox.com/downloads/firmware/mlxup/4.22.1/SFX/linux_x64/mlxup
chmod +x ./mlxup
./mlxup
```

能看到当前网卡的固件情况：

```bash
Querying Mellanox devices firmware ...

Device #1:
----------

  Device Type:      ConnectX3Pro
  Part Number:      764285-B21_Ax
  Description:      HP InfiniBand FDR/Ethernet 10Gb/40Gb 2-port 544+FLR-QSFP Adapter
  PSID:             HP_1380110017
  PCI Device Name:  /dev/mst/mt4103_pci_cr0
  Port1 GUID:       c4346bffffdfe181
  Port2 GUID:       c4346bffffdfe182
  Versions:         Current        Available     
     FW             2.42.5700      N/A           

  Status:           No matching image found
```

### 安装 mstflint

```bash
apt install mstflint
```

## 设置网卡

### 启动 mft 工具

```bash
mst start
```

输出为：

```bash
Starting MST (Mellanox Software Tools) driver set
Loading MST PCI module - Success
Loading MST PCI configuration module - Success
Create devices
Unloading MST PCI module (unused) - Success
```

### 查看网卡配置

```bash
mst status
```

可以看到当前网卡信息（这是只插了一块 mcx4121a 网卡的情况):

```bash
MST modules:
------------
    MST PCI module is not loaded
    MST PCI configuration module loaded

MST devices:
------------
/dev/mst/mt4117_pciconf0         - PCI configuration cycles access.
                                   domain:bus:dev.fn=0000:06:00.0 addr.reg=88 data.reg=92 cr_bar.gw_offset=-1
                                   Chip revision is: 00
```

继续执行命令：

```bash
mlxconfig -d /dev/mst/mt4117_pciconf0 q
```

可以看到网卡的配置信息：

```bash
Device #1:
----------

Device type:        ConnectX4LX         
Name:               MCX4121A-ACU_Ax     
Description:        ConnectX-4 Lx EN network interface card; 25GbE dual-port SFP28; PCIe3.0 x8; UEFI Enabled; tall bracket
Device:             /dev/mst/mt4117_pciconf0

Configurations:                                          Next Boot
        MEMIC_BAR_SIZE                              0                   
        MEMIC_SIZE_LIMIT                            _256KB(1)           
        FLEX_PARSER_PROFILE_ENABLE                  0                   
        FLEX_IPV4_OVER_VXLAN_PORT                   0                   
        ROCE_NEXT_PROTOCOL                          254                 
        PF_NUM_OF_VF_VALID                          False(0)            
        NON_PREFETCHABLE_PF_BAR                     False(0)            
        VF_VPD_ENABLE                               False(0)            
        STRICT_VF_MSIX_NUM                          False(0)            
        VF_NODNIC_ENABLE                            False(0)            
        NUM_PF_MSIX_VALID                           True(1)             
        NUM_OF_VFS                                  8                   
        NUM_OF_PF                                   2                   
        SRIOV_EN                                    True(1)             
        PF_LOG_BAR_SIZE                             5                   
        VF_LOG_BAR_SIZE                             0                   
        NUM_PF_MSIX                                 63                  
        NUM_VF_MSIX                                 11                  
        INT_LOG_MAX_PAYLOAD_SIZE                    AUTOMATIC(0)        
        PCIE_CREDIT_TOKEN_TIMEOUT                   0                   
        ACCURATE_TX_SCHEDULER                       False(0)            
        PARTIAL_RESET_EN                            False(0)            
        SW_RECOVERY_ON_ERRORS                       False(0)            
        RESET_WITH_HOST_ON_ERRORS                   False(0)            
        PCI_BUS0_RESTRICT_SPEED                     PCI_GEN_1(0)        
        PCI_BUS0_RESTRICT_ASPM                      False(0)            
        PCI_BUS0_RESTRICT_WIDTH                     PCI_X1(0)           
        PCI_BUS0_RESTRICT                           False(0)            
        PCI_DOWNSTREAM_PORT_OWNER                   Array[0..15]        
        CQE_COMPRESSION                             BALANCED(0)         
        IP_OVER_VXLAN_EN                            False(0)            
        MKEY_BY_NAME                                False(0)            
        UCTX_EN                                     True(1)             
        PCI_ATOMIC_MODE                             PCI_ATOMIC_DISABLED_EXT_ATOMIC_ENABLED(0)
        TUNNEL_ECN_COPY_DISABLE                     False(0)            
        LRO_LOG_TIMEOUT0                            6                   
        LRO_LOG_TIMEOUT1                            7                   
        LRO_LOG_TIMEOUT2                            8                   
        LRO_LOG_TIMEOUT3                            13                  
        ICM_CACHE_MODE                              DEVICE_DEFAULT(0)   
        TX_SCHEDULER_BURST                          0                   
        LOG_MAX_QUEUE                               17                  
        LOG_DCR_HASH_TABLE_SIZE                     14                  
        MAX_PACKET_LIFETIME                         0                   
        DCR_LIFO_SIZE                               16384               
        ROCE_CC_PRIO_MASK_P1                        255                 
        ROCE_CC_PRIO_MASK_P2                        255                 
        CLAMP_TGT_RATE_AFTER_TIME_INC_P1            True(1)             
        CLAMP_TGT_RATE_P1                           False(0)            
        RPG_TIME_RESET_P1                           300                 
        RPG_BYTE_RESET_P1                           32767               
        RPG_THRESHOLD_P1                            1                   
        RPG_MAX_RATE_P1                             0                   
        RPG_AI_RATE_P1                              5                   
        RPG_HAI_RATE_P1                             50                  
        RPG_GD_P1                                   11                  
        RPG_MIN_DEC_FAC_P1                          50                  
        RPG_MIN_RATE_P1                             1                   
        RATE_TO_SET_ON_FIRST_CNP_P1                 0                   
        DCE_TCP_G_P1                                1019                
        DCE_TCP_RTT_P1                              1                   
        RATE_REDUCE_MONITOR_PERIOD_P1               4                   
        INITIAL_ALPHA_VALUE_P1                      1023                
        MIN_TIME_BETWEEN_CNPS_P1                    4                   
        CNP_802P_PRIO_P1                            6                   
        CNP_DSCP_P1                                 48                  
        CLAMP_TGT_RATE_AFTER_TIME_INC_P2            True(1)             
        CLAMP_TGT_RATE_P2                           False(0)            
        RPG_TIME_RESET_P2                           300                 
        RPG_BYTE_RESET_P2                           32767               
        RPG_THRESHOLD_P2                            1                   
        RPG_MAX_RATE_P2                             0                   
        RPG_AI_RATE_P2                              5                   
        RPG_HAI_RATE_P2                             50                  
        RPG_GD_P2                                   11                  
        RPG_MIN_DEC_FAC_P2                          50                  
        RPG_MIN_RATE_P2                             1                   
        RATE_TO_SET_ON_FIRST_CNP_P2                 0                   
        DCE_TCP_G_P2                                1019                
        DCE_TCP_RTT_P2                              1                   
        RATE_REDUCE_MONITOR_PERIOD_P2               4                   
        INITIAL_ALPHA_VALUE_P2                      1023                
        MIN_TIME_BETWEEN_CNPS_P2                    4                   
        CNP_802P_PRIO_P2                            6                   
        CNP_DSCP_P2                                 48                  
        LLDP_NB_DCBX_P1                             False(0)            
        LLDP_NB_RX_MODE_P1                          OFF(0)              
        LLDP_NB_TX_MODE_P1                          OFF(0)              
        LLDP_NB_DCBX_P2                             False(0)            
        LLDP_NB_RX_MODE_P2                          OFF(0)              
        LLDP_NB_TX_MODE_P2                          OFF(0)              
        ROCE_RTT_RESP_DSCP_P1                       0                   
        ROCE_RTT_RESP_DSCP_MODE_P1                  DEVICE_DEFAULT(0)   
        ROCE_RTT_RESP_DSCP_P2                       0                   
        ROCE_RTT_RESP_DSCP_MODE_P2                  DEVICE_DEFAULT(0)   
        DCBX_IEEE_P1                                True(1)             
        DCBX_CEE_P1                                 True(1)             
        DCBX_WILLING_P1                             True(1)             
        DCBX_IEEE_P2                                True(1)             
        DCBX_CEE_P2                                 True(1)             
        DCBX_WILLING_P2                             True(1)             
        KEEP_ETH_LINK_UP_P1                         True(1)             
        KEEP_IB_LINK_UP_P1                          False(0)            
        KEEP_LINK_UP_ON_BOOT_P1                     False(0)            
        KEEP_LINK_UP_ON_STANDBY_P1                  False(0)            
        DO_NOT_CLEAR_PORT_STATS_P1                  False(0)            
        AUTO_POWER_SAVE_LINK_DOWN_P1                False(0)            
        KEEP_ETH_LINK_UP_P2                         True(1)             
        KEEP_IB_LINK_UP_P2                          False(0)            
        KEEP_LINK_UP_ON_BOOT_P2                     False(0)            
        KEEP_LINK_UP_ON_STANDBY_P2                  False(0)            
        DO_NOT_CLEAR_PORT_STATS_P2                  False(0)            
        AUTO_POWER_SAVE_LINK_DOWN_P2                False(0)            
        NUM_OF_VL_P1                                _4_VLs(3)           
        NUM_OF_TC_P1                                _8_TCs(0)           
        NUM_OF_PFC_P1                               8                   
        VL15_BUFFER_SIZE_P1                         0                   
        NUM_OF_VL_P2                                _4_VLs(3)           
        NUM_OF_TC_P2                                _8_TCs(0)           
        NUM_OF_PFC_P2                               8                   
        VL15_BUFFER_SIZE_P2                         0                   
        DUP_MAC_ACTION_P1                           LAST_CFG(0)         
        SRIOV_IB_ROUTING_MODE_P1                    LID(1)              
        IB_ROUTING_MODE_P1                          LID(1)              
        DUP_MAC_ACTION_P2                           LAST_CFG(0)         
        SRIOV_IB_ROUTING_MODE_P2                    LID(1)              
        IB_ROUTING_MODE_P2                          LID(1)              
        PHY_FEC_OVERRIDE_P1                         DEVICE_DEFAULT(0)   
        PHY_FEC_OVERRIDE_P2                         DEVICE_DEFAULT(0)   
        PF_SD_GROUP                                 0                   
        ROCE_CONTROL                                ROCE_ENABLE(2)      
        PCI_WR_ORDERING                             per_mkey(0)         
        MULTI_PORT_VHCA_EN                          False(0)            
        PORT_OWNER                                  True(1)             
        ALLOW_RD_COUNTERS                           True(1)             
        RENEG_ON_CHANGE                             True(1)             
        TRACER_ENABLE                               True(1)             
        IP_VER                                      IPv4(0)             
        BOOT_UNDI_NETWORK_WAIT                      0                   
        UEFI_HII_EN                                 True(1)             
        BOOT_DBG_LOG                                False(0)            
        UEFI_LOGS                                   DISABLED(0)         
        BOOT_VLAN                                   1                   
        LEGACY_BOOT_PROTOCOL                        PXE(1)              
        BOOT_INTERRUPT_DIS                          False(0)            
        BOOT_LACP_DIS                               True(1)             
        BOOT_VLAN_EN                                False(0)            
        BOOT_PKEY                                   0                   
        DYNAMIC_VF_MSIX_TABLE                       False(0)            
        ADVANCED_PCI_SETTINGS                       False(0)            
        SAFE_MODE_THRESHOLD                         10                  
        SAFE_MODE_ENABLE                            True(1)
```

### 修改网卡配置

```bash
mlxconfig -d /dev/mst/mt4117_pciconf0 set SRIOV_EN=1 NUM_OF_VFS=8
```

对于有多块网卡的机器，可以根据需要决定是否为其中的一块或者多块网卡开启 SR-IOV ，然后需要开启的设置 SRIOV_EN=1 ，不需要开启的设置 SRIOV_EN=0。

重启机器。

## 开启 SR-IOV

### 开启前的检查

查看网卡情况：

```bash
ip addr
```

enp6s0f0np0 和 enp6s0f1np1 为这块 cx4 lx 双头网卡的两个网口，这里我们使用了 enp6s0f1np1：

```bash
2: enp6s0f0np0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether b8:ce:f6:0b:d9:1c brd ff:ff:ff:ff:ff:ff
3: enp6s0f1np1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master vmbr0 state UP group default qlen 1000
    link/ether b8:ce:f6:0b:d9:1d brd ff:ff:ff:ff:ff:ff
```

查看网卡 pci 情况：

```bash
lspci -k | grep -i ethernet
```

可以看到 enp6s0f1np1 这个网卡对应的是 06:00.1 ：

```bash
06:00.0 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
06:00.1 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
```

执行命令查看网卡的 vfs 设置：

```bash
cat /sys/bus/pci/devices/0000:06:00.1/sriov_totalvfs
8
```

### 开启的脚本

新建文件 cx4sriov.service：

```bash
cd /etc/systemd/system
vi cx4sriov.service
```

内容为(注意替换 enp6s0f1np1）：

```properties
[Unit]
Description=Script to enable SR-IOV for cx4 NIC on boot

[Service]
Type=simple

start SR-IOV

ExecStartPre=/usr/bin/bash -c '/usr/bin/echo 8 > /sys/class/net/enp6s0f1np1/device/sriov_numvfs'

set VF MAC

ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set dev enp6s0f1np1 vf 0 mac 00:80:00:00:00:00'
ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set dev enp6s0f1np1 vf 1 mac 00:80:00:00:00:01'
ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set dev enp6s0f1np1 vf 2 mac 00:80:00:00:00:02'
ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set dev enp6s0f1np1 vf 3 mac 00:80:00:00:00:03'
ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set dev enp6s0f1np1 vf 4 mac 00:80:00:00:00:04'
ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set dev enp6s0f1np1 vf 5 mac 00:80:00:00:00:05'
ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set dev enp6s0f1np1 vf 6 mac 00:80:00:00:00:06'
ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set dev enp6s0f1np1 vf 7 mac 00:80:00:00:00:07'

set PF up

ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set enp6s0f1np1 up'

set VF up

ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set enp6s0f1np1v0 up'
ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set enp6s0f1np1v1 up'
ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set enp6s0f1np1v2 up'
ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set enp6s0f1np1v3 up'
ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set enp6s0f1np1v4 up'
ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set enp6s0f1np1v5 up'
ExecStartPre=/usr/bin/bash -c '/usr/bin/ip link set enp6s0f1np1v6 up'
ExecStart=/usr/bin/bash -c '/usr/bin/ip link set enp6s0f1np1v7 up'

Restart=on-failure

[Install]
WantedBy=multi-user.target
```

启用 cx4sriov.service：

```bash
systemctl daemon-reload
systemctl enable cx4sriov.service
reboot
```

重启之后查看网卡信息：

```bash
$ lspci -k | grep -i ethernet

06:00.0 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
06:00.1 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx]
06:01.2 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx Virtual Function]
06:01.3 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx Virtual Function]
06:01.4 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx Virtual Function]
06:01.5 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx Virtual Function]
06:01.6 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx Virtual Function]
06:01.7 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx Virtual Function]
06:02.0 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx Virtual Function]
06:02.1 Ethernet controller: Mellanox Technologies MT27710 Family [ConnectX-4 Lx Virtual Function]
```



```bash
$ ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp6s0f0np0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether b8:ce:f6:0b:d9:1c brd ff:ff:ff:ff:ff:ff
3: enp6s0f1np1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master vmbr0 state UP group default qlen 1000
    link/ether b8:ce:f6:0b:d9:1d brd ff:ff:ff:ff:ff:ff
4: vmbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether b8:ce:f6:0b:d9:1d brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.109/24 scope global vmbr0
       valid_lft forever preferred_lft forever
    inet6 fe80::bace:f6ff:fe0b:d91d/64 scope link 
       valid_lft forever preferred_lft forever
5: enp6s0f1v0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether ae:0a:fe:86:e3:9a brd ff:ff:ff:ff:ff:ff
6: enp6s0f1v1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 12:e3:9e:f9:22:b6 brd ff:ff:ff:ff:ff:ff
7: enp6s0f1v2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 2a:f5:00:14:0f:cc brd ff:ff:ff:ff:ff:ff
8: enp6s0f1v3: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether ba:5d:6c:3d:4a:68 brd ff:ff:ff:ff:ff:ff
9: enp6s0f1v4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 86:e2:c0:ec:a3:d8 brd ff:ff:ff:ff:ff:ff
10: enp6s0f1v5: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether aa:52:93:a7:01:af brd ff:ff:ff:ff:ff:ff
11: enp6s0f1v6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether be:b2:59:79:13:9f brd ff:ff:ff:ff:ff:ff
12: enp6s0f1v7: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 3a:07:60:b5:04:74 brd ff:ff:ff:ff:ff:ff
```





## 参考资料

- [PVE8.0保姆级AIO安装教程 开启网卡SRIOV,跳过虚拟交换机提高网络性能,减少CPU负载 – 小屋 (geekxw.top)](https://www.geekxw.top/?p=638): 另外一个做法。
