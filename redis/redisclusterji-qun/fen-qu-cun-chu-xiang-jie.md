# 分区存储详解

## 参考文档：

官方文档：[http://doc.redisfans.com/topic/cluster-tutorial.html\#id10](http://doc.redisfans.com/topic/cluster-tutorial.html#id10)

## **槽\(slot\)的基本概念** {#autoid-3-0-0}

Redis 集群键分布算法使用数据分片（sharding）而非一致性哈希（consistency hashing）来实现：一个 Redis 集群包含 16384 个哈希槽（hash slot）， 它们的编号为0、1、2、3……16382、16383，这个槽是一个逻辑意义上的槽，实际上并不存在。redis中的每个key都属于这 16384 个哈希槽的其中一个，存取key时都要进行key-&gt;slot的映射计算。

下面我们来看看启动集群时候打印的信息:

```
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.2.128:7031
192.168.2.128:7032
192.168.2.128:7033
Adding replica 192.168.2.128:7034 to 192.168.2.128:7031
Adding replica 192.168.2.128:7035 to 192.168.2.128:7032
Adding replica 192.168.2.128:7036 to 192.168.2.128:7033
M: bee706db5ae182c5be9b9bdf94c2d6f3f8c8ec5c 192.168.2.128:7031
slots:0-5460 (5461 slots) master
M: 72826f06dbf3be163f2f456ca24caed76a15bdf4 192.168.2.128:7032
slots:5461-10922 (5462 slots) master
M: ab6e9d1dfc471225eef01e57be563157f81d26b3 192.168.2.128:7033
slots:10923-16383 (5461 slots) master
......
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

从上面信息可以看出，创建集群的时候，哈希槽被分配到了三个主节点上，从节点是没有哈希槽的。7031负责编号为0-5460 共5461个 slots，7032负责编号为5461-10922共5462 个 slots，7033负责编号为10923-16383 共5461个 slots。

## **键-槽映射算法** {#autoid-3-1-0}

和memcached一样，redis也采用一定的算法进行键-槽（key-&gt;slot）之间的映射。memcached采用一致性哈希（consistency hashing）算法进行键-节点（key-node）之间的映射，而redis集群使用集群公式来计算键 key 属于哪个槽：

HASH\_SLOT（key）=CRC16\(key\) % 16384

其中 CRC16\(key\) 语句用于计算键 key 的 CRC16 校验和 。key经过公式计算后得到所对应的哈希槽，而哈希槽被某个主节点管理，从而确定key在哪个主节点上存取，这也是redis将数据均匀分布到各个节点上的基础。

[![](https://images2015.cnblogs.com/blog/783994/201607/783994-20160718161342169-233368796.png "wps71B9.tmp\[4\]")](http://images2015.cnblogs.com/blog/783994/201607/783994-20160718161341076-206623186.png)

键-槽-节点（key-&gt;slot-&gt;node）映射示意图

## **集群分区好处** {#autoid-3-2-0}

无论是memcached的一致性哈希算法，还是redis的集群分区，最主要的目的都是在移除、添加一个节点时对已经存在的缓存数据的定位影响尽可能的降到最小。redis将哈希槽分布到不同节点的做法使得用户可以很容易地向集群中添加或者删除节点， 比如说：

l如果用户将新节点 D 添加到集群中， 那么集群只需要将节点 A 、B 、 C 中的某些槽移动到节点 D 就可以了。

l与此类似， 如果用户要从集群中移除节点 A ， 那么集群只需要将节点 A 中的所有哈希槽移动到节点 B 和节点 C ， 然后再移除空白（不包含任何哈希槽）的节点 A 就可以了。

因为将一个哈希槽从一个节点移动到另一个节点不会造成节点阻塞， 所以无论是添加新节点还是移除已存在节点， 又或者改变某个节点包含的哈希槽数量， 都不会造成集群下线，从而保证集群的可用性。下面我们就来学习下集群中节点的增加和删除。

