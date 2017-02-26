---
layout: post
title:  docker 和 firewalld
date:   2017-02-26 21:12:08 +0800
---

# docker 和 firewalld

不能一起用。

`docker-engine` 本身的端口隐射实际上是使用 `iptables`.

`firewalld` 有自己的端口管理规则.

这两个规则会冲突。二者只能选其一。


**请使用 `iptables`**

