# RDB与AOF异同比较

## RDB 和 AOF 之间的相互作用

在版本号大于等于 2.4 的 Redis 中，[_BGSAVE_](http://doc.redisfans.com/server/bgsave.html#bgsave)执行的过程中， 不可以执行[_BGREWRITEAOF_](http://doc.redisfans.com/server/bgrewriteaof.html#bgrewriteaof)。 反过来说， 在[_BGREWRITEAOF_](http://doc.redisfans.com/server/bgrewriteaof.html#bgrewriteaof)执行的过程中， 也不可以执行[_BGSAVE_](http://doc.redisfans.com/server/bgsave.html#bgsave)。

这可以防止两个 Redis 后台进程同时对磁盘进行大量的 I/O 操作。

如果[_BGSAVE_](http://doc.redisfans.com/server/bgsave.html#bgsave)正在执行， 并且用户显示地调用[_BGREWRITEAOF_](http://doc.redisfans.com/server/bgrewriteaof.html#bgrewriteaof)命令， 那么服务器将向用户回复一个OK状态， 并告知用户，[_BGREWRITEAOF_](http://doc.redisfans.com/server/bgrewriteaof.html#bgrewriteaof)已经被预定执行： 一旦[_BGSAVE_](http://doc.redisfans.com/server/bgsave.html#bgsave)执行完毕，[_BGREWRITEAOF_](http://doc.redisfans.com/server/bgrewriteaof.html#bgrewriteaof)就会正式开始。

当 Redis 启动时， 如果 RDB 持久化和 AOF 持久化都被打开了， 那么程序会优先使用 AOF 文件来恢复数据集， 因为 AOF 文件所保存的数据通常是最完整的。

