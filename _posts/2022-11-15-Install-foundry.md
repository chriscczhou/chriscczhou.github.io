---
title: "安装foundry框架"
date: 2022-11-15T0:18:30-04:00
categories:
  - Tools
tags:
  - Tools
---

---
title: "安装foundry框架"
date: 2022-11-15T0:18:30-04:00
categories:
  - Tools
tags:
  - Tools
---



昨天试了下安装foundry框架，可太不容易了，在自己Windows云主机上安装遇到了无法解决的报错，后来我又在Centos上安装，结果又遇到了`forge: /lib/x86_64-linux-gnu/libc.so.6: version 'GLIBC_2.29' not found (required by forge)`的报错。官方给出的解决方式有两种：1.从源码开始构建（`foundryup -b master`）;2.使用Docker。Docker暂时作为最后实在没有办法的办法，我尝试了第一种，但是编译极其得慢，可能是我云主机配置比较拉（1核1g的）。从报错看应该是glibc 版本低导致的，所以升级glibc 版本应该就能解决这个问题，正好我也看到Alibaba Cloud Linux 3的操作系统介绍里说了默认搭载了glibc 2.32，那我直接换操作系统就行了。

以下就是安装过程中用到的一些命令：

```
# 1.安装rust环境
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
# 2.下载foundryup
curl -L https://foundry.paradigm.xyz | bash
source /root/.bashrc
# 3.运行foundryup，安装foundry环境
foundryup
```

