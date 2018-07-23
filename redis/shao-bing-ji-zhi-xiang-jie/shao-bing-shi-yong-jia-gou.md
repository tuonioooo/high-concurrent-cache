# 哨兵机制详解（sentinel）

## 哨兵经典架构

![](/assets/import-sentinel-01.png)

## sentinel结构

![](/assets/import-sentinel-02.png)![](/assets/import.png)

## 哨兵工作原理

Sentinel 通过配置文件发现Master，如下：

[![](https://images2015.cnblogs.com/blog/929899/201611/929899-20161130181518927-301322262.png "image")](http://images2015.cnblogs.com/blog/929899/201611/929899-20161130181518131-1097951986.png)[![](https://images2015.cnblogs.com/blog/929899/201611/929899-20161130181519787-2078667359.png "image")](http://images2015.cnblogs.com/blog/929899/201611/929899-20161130181519365-369258435.png)

Sentinel 通过向Master发送 INFO 命令来自动获得所有Slave的地址

跟Master一样，Sentinel 会与每个被发现的Slave创建命令连接和订阅连接

[![](https://images2015.cnblogs.com/blog/929899/201611/929899-20161130181521006-658641784.png "image")](http://images2015.cnblogs.com/blog/929899/201611/929899-20161130181520115-2039756327.png)

Sentinel 会通过命令连接向被监视的主从服务器发送 HELLO 信息，该消息包含 Sentinel 的 IP、端口号、ID 等内容，以此来向其他 Sentinel 宣告自己的存在。与此同时，Sentinel 会通过订阅连接接收其他 Sentinel 的HELLO 信息，以此来发现监视同一个Master的其他 Sentinel

[![](https://images2015.cnblogs.com/blog/929899/201611/929899-20161130181521959-861339059.png "image")](http://images2015.cnblogs.com/blog/929899/201611/929899-20161130181521412-320880559.png)

[![](https://images2015.cnblogs.com/blog/929899/201611/929899-20161130181522709-936612875.png "image")](http://images2015.cnblogs.com/blog/929899/201611/929899-20161130181522412-1901259983.png)

[![](https://images2015.cnblogs.com/blog/929899/201611/929899-20161130181523443-1327331292.png "image")](http://images2015.cnblogs.com/blog/929899/201611/929899-20161130181523052-771911876.png)

[![](https://images2015.cnblogs.com/blog/929899/201611/929899-20161130181524193-1004253236.png "image")](http://images2015.cnblogs.com/blog/929899/201611/929899-20161130181523756-1206981123.png)

Sentinel 之间会互相创建命令连接，用于进行通信。

因为已经有主从服务器作为发送和接收 HELLO 信息的中介，所以 Sentinel之间不会创建订阅连接。

[![](https://images2015.cnblogs.com/blog/929899/201611/929899-20161130181524865-879825249.png "image")](http://images2015.cnblogs.com/blog/929899/201611/929899-20161130181524537-609177434.png)

Sentinel 使用 PING 命令来检测实例的状态：如果实例在指定的时间内没有返回回复，或者返回错误的回复，那么该实例会被 Sentinel 判断为SDOWN（subjectively down,主观下线）。

注意：只有一台Sentinel将服务器标记为SDOWN不会触发Failover。只有一定数量的Sentinel都将该服务器标记为SDOWN后，服务器才会被标记为ODOWN（objectively down，客观下线），这时才会触发Failover过程。

参数down-after-milliseconds指定了 Sentinel 认为服务器已经断线所需的毫秒数。

参数quorum用来控制Sentinel投票数

注意：源码中SDOWN对应PFAIL消息，ODOWN对应FAIL消息

[![](https://images2015.cnblogs.com/blog/929899/201611/929899-20161130181525974-325165535.png "image")](http://images2015.cnblogs.com/blog/929899/201611/929899-20161130181525224-1066446284.png)

Automatic Failover

在下线Master的所有Slave中，选出一个数据状态最接近Master的Slave，选择的条件包括：Slave未下线、主从之间的连接断开时间最短，等等。

Sentinel 向被选中的Slave发送 SLAVEOF NO ONE ，将它升级为Master。

向其他Slave发送 SLAVEOF 命令，让它们复制新的Master。

[![](https://images2015.cnblogs.com/blog/929899/201611/929899-20161130181526662-2136662584.png "image")](http://images2015.cnblogs.com/blog/929899/201611/929899-20161130181526318-737698287.png)

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



