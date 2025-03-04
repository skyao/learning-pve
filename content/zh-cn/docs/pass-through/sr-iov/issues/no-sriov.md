---
title: "不支持开启 SR-IOV"
linkTitle: "不支持 SR-IOV"
weight: 100
date: 2021-01-18
description: >
  在 pve 中遇到的不支持开启 SR-IOV 的情况
---

### 技嘉x99 ud4 主板没有 SR-IOV 设置项

这个主板非常奇怪，bios 选项中找不到 SR-IOV 的设置项，当然，虚拟化支持和 vt-x 是可以的。

后来意外发现：只有在清理 cmos 后第一次启动进入 bios 时，能够在网卡列表中看到当前的所有网卡，其中包括我的 hp544+ 和 realtek 8125b。这是可以进入网卡的配置中找到 SR-IOv 的支持选项，默认是开启的。

但奇怪的是，只要保存 bios，之后再次进入 bios, 就会发现再也无法列出 hp544+  网卡 ， 而 realtek 8125b 网卡有时候有，有没有没有。也就是说只有清理 bios 之后的第一次进入bios能看到网卡列表。反复测试过几次都是这样，不清楚问题所在。

尝试过清理bios后，第一次进入，就把所有的配置都设置好，保存，启动机器，然后再试开启 SR-IOV。这是寄希望于虽然网卡列表不显示但第一次配置的信息还能生效，但很遗憾的失败了。

开启 SR-IOV 导致启动时报错：

```bash
[   13.741520] mlx4_core 0000:02:00.0: DMFS high rate steer mode is: performance optimized for limited rule configuration (static)
[   13.741725] mlx4_core 0000:02:00.0: Enabling SR-IOV with 8 VFs
[   13.741741] mlx4_core 0000:02:00.0: not enough MMIO resources for SR-IOV
[   13.741747] mlx4_core 0000:02:00.0: Failed to enable SR-IOV, continuing without SR-IOV (err = -12)
```

最后，无奈的放弃了在 技嘉x99 ud4 主板上开启  SR-IOV 的尝试，最终选择了性能最佳毛病最少的物理机方案（虚拟机直通方案遇到 IRQ 冲突）。

