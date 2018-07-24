# 排行榜

* ## ZADD

**ZADD key score member \[\[score member\] \[score member\] ...\]**

将一个或多个member元素及其score值加入到有序集key当中。

如果某个member已经是有序集的成员，那么更新这个member的score值，并通过重新插入这个member元素，来保证该member在正确的位置上。

score值可以是整数值或双精度浮点数。

如果key不存在，则创建一个空的有序集并执行[ZADD](http://doc.redisfans.com/sorted_set/zadd.html#zadd)操作。

当key存在但不是有序集类型时，返回一个错误。

对有序集的更多介绍请参见[sorted set](http://redis.io/topics/data-types#sorted-sets)。

在 Redis 2.4 版本以前，[ZADD](http://doc.redisfans.com/sorted_set/zadd.html#zadd)每次只能添加一个元素。

**可用版本：**

&gt;= 1.2.0

**时间复杂度:**

O\(M\*log\(N\)\)， N 是有序集的基数， M 为成功添加的新成员的数量。

**返回值:**

被成功添加的新成员的数量，不包括那些被更新的、已经存在的成员。



* ## ZRANGE

**ZRANGE key start stop \[WITHSCORES\]**

返回有序集key中，指定区间内的成员。

其中成员的位置按score值递增\(从小到大\)来排序。

具有相同score值的成员按字典序\([lexicographical order](http://en.wikipedia.org/wiki/Lexicographical_order)\)来排列。

如果你需要成员按score值递减\(从大到小\)来排列，请使用[_ZREVRANGE_](http://doc.redisfans.com/sorted_set/zrevrange.html#zrevrange)命令。

下标参数 start 和 stop 都以 0 为底，也就是说，以 0 表示有序集第一个成员，以 1 表示有序集第二个成员，以此类推。

你也可以使用负数下标，以 -1 表示最后一个成员， -2 表示倒数第二个成员，以此类推。

超出范围的下标并不会引起错误。

比如说，当 start 的值比有序集的最大下标还要大，或是 start &gt; stop 时， ZRANGE 命令只是简单地返回一个空列表。

另一方面，假如 stop 参数的值比有序集的最大下标还要大，那么 Redis 将 stop 当作最大下标来处理。

可以通过使用 WITHSCORES 选项，来让成员和它的 score 值一并返回，返回列表以 value1,score1, ..., valueN,scoreN 的格式表示。

客户端库可能会返回一些更复杂的数据类型，比如数组、元组等。

可用版本：

&gt;= 1.2.0

时间复杂度:

O\(log\(N\)+M\)， N 为有序集的基数，而 M 为结果集的基数。

返回值:

指定区间内，带有 score 值\(可选\)的有序集成员的列表。



* ## ZADD和ZRANGE应用示例

```
# 添加单个元素

redis> ZADD page_rank 10 google.com
(integer) 1


# 添加多个元素

redis> ZADD page_rank 9 baidu.com 8 bing.com
(integer) 2

redis> ZRANGE page_rank 0 -1 WITHSCORES
1) "bing.com"
2) "8"
3) "baidu.com"
4) "9"
5) "google.com"
6) "10"


# 添加已存在元素，且 score 值不变

redis> ZADD page_rank 10 google.com
(integer) 0

redis> ZRANGE page_rank 0 -1 WITHSCORES  # 没有改变
1) "bing.com"
2) "8"
3) "baidu.com"
4) "9"
5) "google.com"
6) "10"


# 添加已存在元素，但是改变 score 值

redis> ZADD page_rank 6 bing.com
(integer) 0

redis> ZRANGE page_rank 0 -1 WITHSCORES  # bing.com 元素的 score 值被改变
1) "bing.com"
2) "6"
3) "baidu.com"
4) "9"
5) "google.com"
6) "10"
```

```
redis > ZRANGE salary 0 -1 WITHSCORES             # 显示整个有序集成员
1) "jack"
2) "3500"
3) "tom"
4) "5000"
5) "boss"
6) "10086"

redis > ZRANGE salary 1 2 WITHSCORES              # 显示有序集下标区间 1 至 2 的成员
1) "tom"
2) "5000"
3) "boss"
4) "10086"

redis > ZRANGE salary 0 200000 WITHSCORES         # 测试 end 下标超出最大下标时的情况
1) "jack"
2) "3500"
3) "tom"
4) "5000"
5) "boss"
6) "10086"

redis > ZRANGE salary 200000 3000000 WITHSCORES   # 测试当给定区间不存在于有序集时的情况
(empty list or set)
```



