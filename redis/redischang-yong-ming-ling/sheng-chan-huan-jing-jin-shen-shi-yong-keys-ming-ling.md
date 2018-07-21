# 生产环境谨慎使用keys命令

## keys命令的使用 {#一keys命令的使用}

进入redis-cli之后，我们通常比较关心的是有哪些key\(当然也可以用其他客户端工具\)，那么就不得不说keys命令

* **keys命令语法**

?    匹配一个字符

 \*    匹配任意个（包括0个）字符

\[\]    匹配括号间的任一个字符，可以使用 "-" 符号表示一个范围，如 a\[b-d\] 可以匹配 "ab","ac","ad"

\x    匹配字符x，用于转义符号，如果要匹配 "?" 就需要使用 \?

* **获取当前库下的所有key**

```
keys *
```

如下图所示，存在四个key:redis01、redis11、hbase01、hbase11

![](/assets/import-redis-01.png)

* **查找以redis开头的所有key**

```
keys redis*
```

![](/assets/import-redis-02.png)

* **查找以hbase开头、第6位为任一字符、以”1”结尾的key**

```
keys hbase?1
```

![](/assets/import-redis-04.png)

* **\[\]，这和正则表示中的\[\]类似，每次可以配其中任何一个字符。**

```
keys redis[01]1
```

> 会匹配”redis01”和”redis11”,“2”不再【01】中，所有”redis21”未能匹配上

![](/assets/import-redis-03.png)

## 利用keys的模糊匹配带来的性能问题 {#二利用keys的模糊匹配带来的性能问题}

从上面开来，keys的模糊匹配功能很方便也很强大，但是在生产环境需要慎用！开发中使用keys的模糊匹配却发现redis的CPU使用率极高，所以公司的redis生产环境将keys命令禁用了！

那怎么解决这种类似的keys模糊匹配问题呢？其中常见的方法就是设置一个set，将需要使用的keys存储在set中。



