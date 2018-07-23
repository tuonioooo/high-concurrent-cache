# 架构分析

### redis-cluster架构图

### ![](/assets/import-rediscluster-01.png)

> 架构细节:
>
> \(1\)所有的redis节点彼此互联\(PING-PONG机制\),内部使用二进制协议优化传输速度和带宽.
>
> \(2\)节点的fail是通过集群中超过半数的master节点检测失效时才生效.
>
> \(3\)客户端与redis节点直连,不需要中间proxy层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可
>
> \(4\)redis-cluster把所有的物理节点映射到\[0-16383\]slot上,cluster 负责维护node&lt;-&gt;slot&lt;-&gt;key

### redis-cluster选举、容错

![](/assets/import-rediscluster-02.png)

> \(1\)选举过程是集群中所有master参与,如果半数以上master节点与故障节点通信超过\(cluster-node-timeout\),认为该节点故障，自动触发故障转移操作.
>
> \(2\):什么时候整个集群不可用\(cluster\_state:fail\)?
>
> a:如果集群任意master挂掉,且当前master没有slave.集群进入fail状态,也可以理解成集群的slot映射\\[0-16383\\]不完整时进入fail状态.
>
> 注意 ：redis-3.0.0.rc1加入cluster-require-full-coverage参数,默认关闭,打开集群兼容部分失败.
>
> b:如果集群超过半数以上master挂掉，无论是否有slave集群进入fail状态.
>
> 注意：当集群不可用时,所有对集群的操作做都不可用，收到\(\(error\) CLUSTERDOWN The cluster is down\)错误

## 详细内容

官方文档：[http://doc.redisfans.com/topic/cluster-tutorial.html](http://doc.redisfans.com/topic/cluster-tutorial.html)

