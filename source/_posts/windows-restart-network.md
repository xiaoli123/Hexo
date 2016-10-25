---
title: windows重新启用网络
date: 2016-10-21
tags:
    - windows
categories: windows
---

> 公司的网络有两个网段，每天来了动态分配。一个快，一个慢。一般遇到慢的我就拔下网线重试。但是这样确实麻烦，而且对插口不太好。所以干脆找个命令来解决这个问题吧！

## 命令：
```
ipconfig/release
ipconfig/renew
```
so easy！妈妈再也不用担心我的网络了！