# RDB详解

## 概述

_**RDB 持久化可以在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）。**_

## 场景

Redis是一个键值对数据库服务器，服务器中通常包含着任意个非空数据库，而每个非空数据库中又可以包含任意个键值对，为了方便起见，我们将服务器中的非空数据库以及它们的键值对统称为数据库状态

举个例子，图10-1 展示了一个包含三个非空数据库的Redis 服务器，这三个数据库以及数据库中的键值对就是该服务器的数据库状态

![](https://images2015.cnblogs.com/blog/1067264/201612/1067264-20161201170648849-476855007.png)

因为Redis 是内存数据库，它将自己的数据库状态储存在内存里面，所以如果不想办法将储存在内存中的数据库状态保存到磁盘里面，那么一旦服务器进程退出，服务器中的数据库状态也会消失不见

为了解决这个问题，Redis提供了RDB持久化功能，这个功能可以将Redis 在内存中的数据库状态保存到磁盘里面，避免数据意外丢失  
RDB持久化既可以手动执行，也可以根据服务器配置选项定期执行，该功能可以将某个时间点上的数据库状态保存到一个RDB文件中，如图10-2所示  
RDB 持久化功能所生成的RDB 文件是一个经过压缩的二进制文件，通过该文件可以还原生成RDB 文件时的数据库状态，如图1 0-3所示

![](https://images2015.cnblogs.com/blog/1067264/201612/1067264-20161201171039865-192573692.png)

因为RDB文件是保存在硬盘里面的，所以即使Redis服务器进程退出，甚至运行Redis服务器的计算机停机，但只要RDB文件仍然存在，Redis服务器就可以用它来还原数据库状态

## RDB的优点

* RDB 是一个非常紧凑（compact）的文件，它保存了 Redis 在某个时间点上的数据集。 这种文件非常适合用于进行备份： 比如说，你可以在最近的 24 小时内，每小时备份一次 RDB 文件，并且在每个月的每一天，也备份一个 RDB 文件。 这样的话，即使遇上问题，也可以随时将数据集还原到不同的版本。

* RDB 非常适用于灾难恢复（disaster recovery）：它只有一个文件，并且内容都非常紧凑，可以（在加密后）将它传送到别的数据中心，或者亚马逊 S3 中。

* RDB 可以最大化 Redis 的性能：父进程在保存 RDB 文件时唯一要做的就是fork出一个子进程，然后这个子进程就会处理接下来的所有保存工作，父进程无须执行任何磁盘 I/O 操作。

* RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。

## RDB 的缺点

* 如果你需要尽量避免在服务器故障时丢失数据，那么 RDB 不适合你。 虽然 Redis 允许你设置不同的保存点（save point）来控制保存 RDB 文件的频率， 但是， 因为RDB 文件需要保存整个数据集的状态， 所以它并不是一个轻松的操作。 因此你可能会至少 5 分钟才保存一次 RDB 文件。 在这种情况下， 一旦发生故障停机， 你就可能会丢失好几分钟的数据。

* 每次保存 RDB 的时候，Redis 都要fork\(\)出一个子进程，并由子进程来进行实际的持久化工作。 在数据集比较庞大时，  
  fork\(\)可能会非常耗时，造成服务器在某某毫秒内停止处理客户端； 如果数据集非常巨大，并且 CPU 时间非常紧张的话，那么这种停止时间甚至可能会长达整整一秒。 虽然 AOF 重写也需要进行fork\(\)，但无论 AOF 重写的执行间隔有多长，数据的耐久性都不会有任何损失。

> 在数据集比较庞大时、并且 CPU 时间非常紧张的话，RDB可能会造成数据损失）

## RDB 快照文件的创建与载入

在默认情况下， Redis 将数据库快照保存在名字为dump.rdb的二进制文件中。

你可以对 Redis 进行设置， 让它在“N秒内数据集至少有M个改动”这一条件被满足时， 自动保存一次数据集。

你也可以通过调用[_SAVE_](http://doc.redisfans.com/server/save.html#save)或者[_BGSAVE_](http://doc.redisfans.com/server/bgsave.html#bgsave)， 手动让 Redis 进行数据集保存操作。

比如说， 以下设置会让 Redis 在满足“60秒内有至少有1000个键被改动”这一条件时， 自动保存一次数据集：

```
save 60 1000
```

这种持久化方式被称为快照（snapshot）。

> 注意：
>
> 创建RDB文件的实际工作由rdb.c/rdbSave函数完成，SAVE命令和BGSAVE命令会以不同的方式调用这个函数，通过以下伪代码可以明显地看出这两个命令之间的区别

```
def SAVE() 
#创建RDB 文件
rdbSave()
def BGSAVE()
#创建子进程
pid = fork()
if pid==0
#子进程负责创建RDB 文件
rdbSave()
#完成之后向父进程发送信号
signal_parent ()
elif pid>0
#父进程继续处理命令请求，并通过轮询等待子进程的信号
handle_request_and_wait_signal()
else
#处理出错情况
handle_fork_error()
```

> 和使用SAVE命令或者BGSAVE命令创建RDB文件不同，RDB 文件的载入工作是在服务启动时自动执行的，所以Redis并没有专门用于载入RDB文件的命令，只要Redis服务器在启动时检测到RDB文件存在，它就会自动载入RDB文件以下是Redis服务器启动时打印的日志记录，其中第二条日志DB loaded from disk就是服务器在成功载入RDB 文件之后打印的

```
$ redis-server
[7379] 30 Aug 21:07:01.270 # Server started, Redis version 2.9.11
[7379] 30 Aug 21:07:01.289 * DB loaded from disk : 0.018 seconds
[7379] 30 Aug 21:07:01.289 * Tbe server is now ready to accept connections on port 6379
```

## 快照的运作方式

当 Redis 需要保存dump.rdb文件时， 服务器执行以下操作：

1. Redis 调用fork\(\)，同时拥有父进程和子进程。
2. 子进程将数据集写入到一个临时 RDB 文件中。
3. 当子进程完成对新 RDB 文件的写入时，Redis 用新 RDB 文件替换原来的 RDB 文件，并删除旧的 RDB 文件。

这种工作方式使得 Redis 可以从写时复制（copy-on-write）机制中获益。

## SAVE命令执行时的服务器状态

前面提到过，当SAVE命令执行时，Redis服务器会被阻塞，所以当SAVE命令正在执行时，客户端发送的所有命令请求都会被拒绝只有在服务器执行完SAVE命令、重新开始接受命令请求之后，客户端发送的命令才会被处理。

## BGSAVE命令执行时的服务器状态

因为BGSAVE命令的保存工作是由子进程执行的，所以在子进程创建RDB文件的过程中，Redis服务器仍然可以继续处理客户端的命令请求，但是，在BGSAVE命令执行期间，服务器处理SAVE、BGSAVE、BGREWRITEAOF三个命令的方式会和平时有所不同

首先，在BGSAVE命令执行期间，客户端发送的SAVE命令会被服务器拒绝，服务器禁止SAVE命令和BGSAVE命令同时执行是为了避免父进程\(服务器进程\)和子进程同时执行两个rdbSave调用，防止产生竞争条件

其次，在BGSAVE命令执行期间，客户端发送的BGSAVE命令会被服务器拒绝，因为同时执行两个BGSAVE 命令也会产生竞争条件

最后，BGREWRITEAOF和BGSAVE两个命令不能同时执行如果BGSAVE命令正在执行，那么客户端发送的BGREWRITEAOF命令会被延迟到BGSAVE命令执行完毕之后执行如果BGREWRITEAOF命令正在执行，那么客户端发送的BGSAVE命令会被服务器拒绝，因为BGREWRITEAOF和BGSAVE两个命令的实际工作都由子进程执行，所以这两个命令在操作方面并没有什么冲突的地方，不能同时执行它们只是一个性能方面的考虑一一并发出两个子进程，井且这两个子进程都同时执行大量的磁盘写入操作，这怎么想都不会是一个好主意。

## RDB 文件载入时的服务器状态

服务器在载入RDB 文件期间，会一直处于阻塞状态，直到载入工作完成为止。

## 自动间隔性保存

因为BGSAVE命令可以在不阻塞服务器进程的情况下执行，所以Redis允许用户通过设置服务器配置的save选项，让服务器每隔一段时间自动执行一次BGSAVE命令

用户可以通过save选项设置多个保存条件，但只要其中任意一个条件被满足，服务器就会执行BGSAVE命令  
举个例子，如果我们向服务器提供以下配置

```
save 900 1
save 300 10
save 60 10000
```

那么只要满足以下三个条件中的任意一个，BGSAVE命令就会被执行  
服务器在900秒之内，对数据库进行了至少1次修改

服务器在300秒之内，对数据库进行了至少10次修改

服务器在60秒之内，对数据库进行了至少10000次修改

举个例子，以下是Redis服务器在60秒之内，对数据库进行了至少10000次修改之后，服务器自动执行BGSAVE命令时打印出来的日志:

```
[5085] 03 Sep 17:09:49.463 * 10000 changes in 60 seconds . Saving ...
[5085] 03 Sep 17:09:49.463 * Background saving started by pid 5189
[5189] 03 Sep 17:09:49.522 • DB saved on disk
[5189] 03 Sep 17:09:49.522 • RDB: 0 MB of memory used by copy-on-write
[5085] 03 Sep 17:09:49.563 • Background saving terminated with succes
```

## 设置保存条件

当Redis服务器启动时，用户可以通过指定配置文件或者传人启动参数的方式设置save选项，如果用户没有主动设置save选项，那么服务帮会为save选项设置默认条件:

```
save 900 1
save 300 10
save 60 10000
```

接着，服务器程序会根据save选项所设置的保存条件，设置服务器状态redisServer结构的saveparams属性:

```
struct redisServer {
//...
//记录了保存条件的数组
struct saveparam *saveparams;
//...
};
```

saveparams属性是一个数组，数组中的每个元素都是一个saveparam结构，每个saveparam结构都保存了一个save选项设置的保存条件:

```
struct saveparam {
//秒数
time_t seconds;
//修改数
int changes;
};
```

比如说，如果save选项的值为以下条件:

```
save 900 1
save 300 10
save 60 10000
```

那么服务器状态中的saveparams数组将会是图10-6所示的样子。

## ![](/assets/import-rdb-01.png)dirty计数器和lastsave属性

除了saveparams数组之外，服务器状态还维持着一个dirty计数器，以及一个lastsave属性:

dirty计数器记录距离上一次成功执行SAVE命令或者BGSAVE命令之后，服务器对数据库状态\(服务器中的所有数据库\)进行了多少次修改\(包括写入、删除、更新等操作\)。  
lastsave属性是一个UNIX时间戳，记录了服务器上一次成功执行SAVE命令或者BGSAVE命令的时间。

当服务器成功执行一个数据库修改命令之后，程序就会对dirty计数器进行更新:命令修改了多少次数据库.dirty计数器的值就增加多少。  
例如，如果我们为一个字符串键设置值:

```
redis>SET message "hello"
OK
```

那么程序会将dirty计数器的值增加1。又例如，如果我们向一个集合键增加三个新元素:

```
redis>SADD database Redis MongoDB MariaDB
(integer) 3
```

那么程序会将dirty计数器的值增加3.

