---
title: "修改locale设置"
linkTitle: "设置locale"
weight: 15
date: 2021-01-18
description: >
  修改locale设置避免报错
---

### 修改配置

默认安装后，有时会遇到 locale 报错：

```bash
perl: warning: Setting locale failed.
perl: warning: Please check that your locale settings:
	LANGUAGE = (unset),
	LC_ALL = (unset),
	LC_IDENTIFICATION = "zh_CN.UTF-8",
	LC_NUMERIC = "zh_CN.UTF-8",
	LC_TIME = "zh_CN.UTF-8",
	LC_PAPER = "zh_CN.UTF-8",
	LC_MONETARY = "zh_CN.UTF-8",
	LC_TELEPHONE = "zh_CN.UTF-8",
	LC_MEASUREMENT = "zh_CN.UTF-8",
	LC_NAME = "zh_CN.UTF-8",
	LC_ADDRESS = "zh_CN.UTF-8",
	LANG = "en_US.UTF-8"
    are supported and installed on your system.
perl: warning: Falling back to a fallback locale ("en_US.UTF-8").
```

最简单的修改方案：

```bash
vi /etc/default/locale
```

将内容修改为：

```properties
LC_CTYPE="en_US.UTF-8"
LC_ALL="en_US.UTF-8"
LANG="en_US.UTF-8"
```

### 参考资料

- https://www.frytea.com/archives/609/
- https://www.xh86.me/?p=1541