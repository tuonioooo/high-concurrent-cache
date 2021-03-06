# 哨兵使用架构（sentinel）

## 概述

Sentinel（哨兵）是Redis 的高可用性解决方案：由一个或多个Sentinel 实例 组成的Sentinel 系统可以监视任意多个主服务器，以及这些主服务器属下的所有从服务器，并在被监视的主服务器进入下线状态时，自动将下线主服务器属下的某个从服务器升级为新的主服务器。

例如：

![](https://images2015.cnblogs.com/blog/1053081/201612/1053081-20161230154322726-1942512342.png)

在Server1 掉线后：

![](https://images2015.cnblogs.com/blog/1053081/201612/1053081-20161230154348742-1055330295.png)

升级Server2 为新的主服务器：

![](https://images2015.cnblogs.com/blog/1053081/201612/1053081-20161230154428320-1980223016.png)

## 哨兵经典架构

![](/assets/import-sentinel-01.png)

## sentinel结构

![](/assets/import-sentinel-02.png)![](/assets/import.png)

## 

## Sentinel功能

Redis 的 Sentinel 系统用于管理多个 Redis 服务器（instance）， 该系统执行以下三个任务：

* **监控（Monitoring）**： Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
* **提醒（Notification）**： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
* **自动故障迁移（Automatic failover）**： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

Redis Sentinel 是一个分布式系统， 你可以在一个架构中运行多个 Sentinel 进程（progress）， 这些进程使用流言协议（gossip protocols\)来接收关于主服务器是否下线的信息， 并使用投票协议（agreement protocols）来决定是否执行自动故障迁移， 以及选择哪个从服务器作为新的主服务器。

虽然 Redis Sentinel 释出为一个单独的可执行文件redis-sentinel， 但实际上它只是一个运行在特殊模式下的 Redis 服务器， 你可以在启动一个普通 Redis 服务器时通过给定--sentinel选项来启动 Redis Sentinel 。

> Redis Sentinel 目前仍在开发中， 这个文档的内容可能随着 Sentinel 实现的修改而变更。
>
> Redis Sentinel 兼容 Redis 2.4.16 或以上版本， 推荐使用 Redis 2.8.0 或以上的版本。



