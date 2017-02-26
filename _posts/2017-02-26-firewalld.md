---
layout: post
title:  firewalld
date:   2017-02-23 19:12:08 +0800
---

# firewalld

`centos 7` 开始使用  `firewall` 来替代之前的的 `iptables`。简单学习后记录部分笔记：

## firewalld 和 iptables 的区别

实际上, `firewalld` 和 `iptables` 都是上层的配置, 都是 `netfilter` 的不同形式的封装。归根结底还是发送命令给 Linux Kernel 的 `netfilter`，让其对不同类型、端口、目标、请求等等形式的包进行处理。

实际上，你也可以卸载 `firewalld` 重新安装  `iptables` 来管理防火墙。

## firewalld 简单使用

### 启动、关闭服务

`centos 7` 开始，`init` 程序使用 `systemd` 替换了 `upstart`. 我们可以使用 `systemd` 来启动、关闭 `firewalld` 服务:

```
# 启动服务
$ sudo systemctl start firewalld.service
# 关闭服务
$ sudo systemctl stop firewalld.service
```

### zone 说明

`firewalld` 本身设定了一个概念，叫做 `zone`. 不同的网卡设备可以属于不用的 `zone` 下。然后不同的 `zone` 又可以设定不同的端口、服务的通行（响应）状态。

如下为 `firewalld` 默认的几个 zone:

| 名称 | 说明 |
| ---- | ---- |
| 阻塞区域（block） | 任何传入的网络数据包都将被阻止 |
| 工作区域（work）| 相信网络上的其他计算机，不会损害你的计算机 |
| 家庭区域（home）| 相信网络上的其他计算机，不会损害你的计算机 |
| 公共区域（public）| 不相信网络上的任何计算机，只有选择接受传入的网络连接 |
| 隔离区域（DMZ）| 隔离区域也称为非军事区域，内外网络之间增加的一层网络，起到缓冲作用。对于隔离区域，只有选择接受传入的网络连接 |
| 信任区域（trusted）| 所有的网络连接都可以接受 |
| 丢弃区域（drop）| 任何传入的网络连接都被拒绝 |
| 内部区域（internal）| 信任网络上的其他计算机，不会损害你的计算机。只有选择接受传入的网络连接 |
| 外部区域（external）| 不相信网络上的其他计算机，不会损害你的计算机。只有选择接受传入的网络连接 |

不同的网卡我们可以设置不同的 `zone`, 如下:

```
$ sudo firewall-cmd --zone=public --change-interface=eth0
# 我们给 eth0 这个 interface 设置 zone 为 public

$ sudo firewall-cmd --get-zone-of-interface=eth0
public
# 反馈为 public 这个 zone
```

### zone 上的操作

#### 增加、删除、查看端口（或服务）

```
$ sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
success 
# 给 public 这个 zone 上创建一个 tcp 类型端口号为 8080 的访问规则（允许访问），并且设置该规则长久有效.
# success 表示生效

$ sudo firewall-cmd --zone=public --remove-port=8080/tcp --permanent
success
# 删除 public 这个 zone 上的一个 tcp 类型端口号为 8080 的访问规则（以后不允许了），如果该规则为长期有效，那么把它从长期有效中删除
# success 表示生效

$ sudo firewall-cmd --zone=public --query-port=8080/tcp --permanent
yes
# 检查 public 这个 zone 上是否有一个长期有效的端口号为  8080 协议为 tcp 的端口访问规则。
# yes 表示有，no 表示没有
```

如果是 **服务**(service) ， 那么如上命令的 `port` 全部修改为 `service` 即可。

**注意**， 为了让如上命令生效，如要执行 `firewall-cmd --reload`。
