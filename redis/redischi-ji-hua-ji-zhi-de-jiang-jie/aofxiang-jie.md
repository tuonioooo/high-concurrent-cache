# AOF详解

## 概述

持久化记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集。 AOF 文件中的命令全部以 Redis 协议的格式来保存，新命令会被追加到文件的末尾。 Redis 还可以在后台对 AOF 文件进行重写（rewrite），使得 AOF 文件的体积不会超出保存数据集状态所需的实际大小。

## AOF 的优点

使用 AOF 持久化会让 Redis 变得非常耐久（much more durable）：你可以设置不同的

* fsync
  策略，比如无
  fsync
  ，每秒钟一次
  fsync
  ，或者每次执行写入命令时
  fsync
  。 AOF 的默认策略为每秒钟
  fsync
  一次，在这种配置下，Redis 仍然可以保持良好的性能，并且就算发生故障停机，也最多只会丢失一秒钟的数据（
  fsync
  会在后台线程执行，所以主线程可以继续努力地处理命令请求）。
* AOF 文件是一个只进行追加操作的日志文件（append only log）， 因此对 AOF 文件的写入不需要进行
  seek
  ， 即使日志因为某些原因而包含了未写入完整的命令（比如写入时磁盘已满，写入中途停机，等等），
  redis-check-aof
  工具也可以轻易地修复这种问题。
* Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写： 重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。
* AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。 导出（export） AOF 文件也非常简单： 举个例子， 如果你不小心执行了
  [_FLUSHALL_](http://doc.redisfans.com/server/flushall.html#flushall)
  命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的
  [_FLUSHALL_](http://doc.redisfans.com/server/flushall.html#flushall)
  命令， 并重启 Redis ， 就可以将数据集恢复到
  [_FLUSHALL_](http://doc.redisfans.com/server/flushall.html#flushall)
  执行之前的状态。



