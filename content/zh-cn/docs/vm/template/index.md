---
title: "模板转换"
linkTitle: "模板转换"
weight: 100
date: 2025-05-09
description: >
  在虚拟机和模板之间转换
---

## 虚拟机转模板

这个操作非常简单，只需要在虚拟机详情页面点击“转换为模板”按钮即可。

## 模板转虚拟机

一般做这个操作的原因都是为了对模板内容进行更新，之后会重新转换回模板。因为模板是无法启动，无法修改的，所以只能先将模板转成虚拟机，完成修改后，再转换回模板。

但在这个过程中，vm id 会发生修改，如果需要固定使用某个 vm id，比如我在制作通用模板时 vm id 会有命名要求，有些一些小的改动不适合重新分配一个新的id。

这样就需要保证改动前后的 vm id 一致。

### 克隆-删除-再克隆

这个方案操作比较麻烦，需要多次磁盘操作，会有大量的磁盘读写，很慢，但是安全，而且只要界面上点几次，就可以完成转换：

- 从模板克隆虚拟机，用一个 临时 vm id=1001
- 修改虚拟机
- 删除模板：主要是腾出 vm id=990201
- 从虚拟机克隆一份新的虚拟机：用之前腾出来的vm id=990201，注意要用 full clone，不能用 linked clone
- 删除临时虚拟机 vm id=1001
- 将第二个虚拟机转换为模板

### 直接转换

这个方案需要使用到 qm 命令，要在命令行进行操作，没有图形界面：

```bash
qm set 990201 --template 0
```

此时界面上模板就转为虚拟机了，但是这个虚拟机无法直接启动，会报错如下：

```bash
kvm: -drive if=pflash,unit=1,id=drive-efidisk0,format=qcow2,file=/var/lib/vz/images/990201/base-990201-disk-0.qcow2: Could not open '/var/lib/vz/images/990201/base-990201-disk-0.qcow2': Operation not permitted
TASK ERROR: start failed: QEMU exited with code 1
```

此时进入这个虚拟机目录，会发现文件权限有问题，都是只读，所以虚拟机启动失败：

```bash
$ cd /var/lib/vz/images/990201
$ ls -la                           
total 43357352
drwxr----- 2 root root         4096 May  9 09:58 .
drwxr-xr-x 6 root root         4096 May  8 10:05 ..
-r--r--r-- 1 root root       917504 May  8 10:07 base-990201-disk-0.qcow2
-r--r--r-- 1 root root 549839962112 May  9 09:57 base-990201-disk-1.qcow2
```

需要修改权限：

```bash
chmod 600 /var/lib/vz/images/990201/base-990201-disk-0.qcow2
chmod 600 /var/lib/vz/images/990201/base-990201-disk-1.qcow2
```

但直接修改会报错，即使是 root 帐号也不行：

```bash
chmod: changing permissions of '/var/lib/vz/images/990201/base-990201-disk-0.qcow2': Operation not permitted
```

也是因为虚拟机转成模板时，这两个文件不仅仅被修改为只读，而且文件被标记为不可变（Immutable）：

```bash
$ lsattr /var/lib/vz/images/990201/base-990201-disk-0.qcow2
----i--------e-- /var/lib/vz/images/990201/base-990201-disk-0.qcow2
```

需要使用 `chattr -i` 命令去掉不可变属性：

```bash
chattr -i /var/lib/vz/images/990201/base-990201-disk-0.qcow2
chattr -i /var/lib/vz/images/990201/base-990201-disk-1.qcow2
```

此时再修改权限，就可以成功了：

```bash
chmod 600 /var/lib/vz/images/990201/base-990201-disk-0.qcow2
chmod 600 /var/lib/vz/images/990201/base-990201-disk-1.qcow2
```

此时再启动虚拟机，就可以正常启动了。修改完成之后，再转为模板即可。



