# RDB与AOF异同比较

## RDB 和 AOF 之间的相互作用

在版本号大于等于 2.4 的 Redis 中，[_BGSAVE_](http://doc.redisfans.com/server/bgsave.html#bgsave)执行的过程中， 不可以执行[_BGREWRITEAOF_](http://doc.redisfans.com/server/bgrewriteaof.html#bgrewriteaof)。 反过来说， 在[_BGREWRITEAOF_](http://doc.redisfans.com/server/bgrewriteaof.html#bgrewriteaof)执行的过程中， 也不可以执行[_BGSAVE_](http://doc.redisfans.com/server/bgsave.html#bgsave)。

这可以防止两个 Redis 后台进程同时对磁盘进行大量的 I/O 操作。

如果[_BGSAVE_](http://doc.redisfans.com/server/bgsave.html#bgsave)正在执行， 并且用户显示地调用[_BGREWRITEAOF_](http://doc.redisfans.com/server/bgrewriteaof.html#bgrewriteaof)命令， 那么服务器将向用户回复一个OK状态， 并告知用户，[_BGREWRITEAOF_](http://doc.redisfans.com/server/bgrewriteaof.html#bgrewriteaof)已经被预定执行： 一旦[_BGSAVE_](http://doc.redisfans.com/server/bgsave.html#bgsave)执行完毕，[_BGREWRITEAOF_](http://doc.redisfans.com/server/bgrewriteaof.html#bgrewriteaof)就会正式开始。

当 Redis 启动时， 如果 RDB 持久化和 AOF 持久化都被打开了， 那么程序会优先使用 AOF 文件来恢复数据集， 因为 AOF 文件所保存的数据通常是最完整的。

## RDB 和 AOF ，我应该用哪一个？

一般来说， 如果想达到足以媲美 PostgreSQL 的数据安全性， 你应该同时使用两种持久化功能。

如果你非常关心你的数据， 但仍然可以承受数分钟以内的数据丢失， 那么你可以只使用 RDB 持久化。

有很多用户都只使用 AOF 持久化， 但我们并不推荐这种方式： 因为定时生成 RDB 快照（snapshot）非常便于进行数据库备份， 并且 _**RDB 恢复数据集的速度也要比 AOF 恢复的速度要快**_， 除此之外， 使用 RDB 还可以避免之前提到的 AOF 程序的 bug 。

因为以上提到的种种原因， 未来我们可能会将 AOF 和 RDB 整合成单个持久化模型。 （这是一个长期计划。）

接下来的几个小节将介绍 RDB 和 AOF 的更多细节。

## 怎么从 RDB 持久化切换到 AOF 持久化

在 Redis 2.2 或以上版本，可以在不重启的情况下，从 RDB 切换到 AOF ：

1. 为最新的dump.rdb文件创建一个备份。
2. 将备份放到一个安全的地方。
3. 执行以下两条命令：
4. > ```
   > redis-cli> CONFIG SET appendonly yes
   >
   > redis-cli> CONFIG SET save ""
   > ```
5. 确保命令执行之后，数据库的键的数量没有改变。
6. 确保写命令会被正确地追加到 AOF 文件的末尾。

步骤 3 执行的第一条命令开启了 AOF 功能： Redis 会阻塞直到初始 AOF 文件创建完成为止， 之后 Redis 会继续处理命令请求， 并开始将写入命令追加到 AOF 文件末尾。

步骤 3 执行的第二条命令用于关闭 RDB 功能。 这一步是可选的， 如果你愿意的话， 也可以同时使用 RDB 和 AOF 这两种持久化功能。

别忘了在redis.conf中打开 AOF 功能！ 否则的话， 服务器重启之后， 之前通过CONFIGSET设置的配置就会被遗忘， 程序会按原来的配置来启动服务器。

译注： 原文这里还有介绍 2.0 版本的切换方式， 考虑到 2.0 已经很老旧了， 这里省略了对那部分文档的翻译， 有需要的请参考原文。

