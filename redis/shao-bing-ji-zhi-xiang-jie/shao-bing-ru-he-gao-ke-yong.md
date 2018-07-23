# 哨兵如何高可用

参考文档：

官方文档：[http://doc.redisfans.com/topic/sentinel.html](http://doc.redisfans.com/topic/sentinel.html)

Redis SentinelSentinel\(哨兵\)是用于监控redis集群中Master状态的工具，其已经被集成在redis2.4+的版本中。

一、Sentinel作用：

1\)：Master状态检测

2\)：如果Master异常，则会进行Master-Slave切换，将其中一个Slave作为Master，将之前的Master作为Slave

3\)：Master-Slave切换后，master\_redis.conf、slave\_redis.conf和sentinel.conf的内容都会发生改变，即master\_redis.conf中会多一行slaveof的配置，sentinel.conf的监控目标会随之调换

二、Sentinel工作方式：

1\)：每个Sentinel以每秒钟一次的频率向它所知的Master，Slave以及其他 Sentinel 实例发送一个 PING 命令

2\)：如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值， 则这个实例会被 Sentinel 标记为主观下线。

3\)：如果一个Master被标记为主观下线，则正在监视这个Master的所有 Sentinel 要以每秒一次的频率确认Master的确进入了主观下线状态。

4\)：当有足够数量的 Sentinel（大于等于配置文件指定的值）在指定的时间范围内确认Master的确进入了主观下线状态， 则Master会被标记为客观下线

5\)：在一般情况下， 每个 Sentinel 会以每 10 秒一次的频率向它已知的所有Master，Slave发送 INFO 命令。

6\)：当Master被 Sentinel 标记为客观下线时，Sentinel 向下线的 Master 的所有 Slave 发送 INFO 命令的频率会从 10 秒一次改为每秒一次

7\)：若没有足够数量的 Sentinel 同意 Master 已经下线， Master 的客观下线状态就会被移除。

若 Master 重新向 Sentinel 的 PING 命令返回有效回复， Master 的主观下线状态就会被移除。

主观下线和客观下线

**主观下线**：Subjectively Down，简称 SDOWN，指的是当前 Sentinel 实例对某个redis服务器做出的下线判断。

**客观下线**：Objectively Down， 简称 ODOWN，指的是多个 Sentinel 实例在对Master Server做出 SDOWN 判断，并且通过 SENTINEL is-master-down-by-addr 命令互相交流之后，得出的Master Server下线判断，然后开启failover.

**SDOWN**

适合于Master和Slave，只要一个 Sentinel 发现Master进入了ODOWN， 这个 Sentinel 就可能会被其他 Sentinel 推选出， 并对下线的主服务器执行自动故障迁移操作。

**ODOWN**

只适用于Master，对于Slave的 Redis 实例，Sentinel 在将它们判断为下线前不需要进行协商， 所以Slave的 Sentinel 永远不会达到ODOWN。

