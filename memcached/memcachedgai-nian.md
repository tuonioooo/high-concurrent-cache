# Memcached概念

## 概述

Memcached 是一个高性能的分布式内存对象缓存系统，用于动态Web应用以减轻数据库负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高动态、数据库驱动网站的速度。Memcached基于一个存储键/值对的[hashmap](https://baike.baidu.com/item/hashmap/1167707)。其[守护进程](https://baike.baidu.com/item/守护进程/966835)（daemon ）是用[C](https://baike.baidu.com/item/C/7252092)写的，但是客户端可以用任何语言来编写，并通过memcached协议与守护进程通信。

## 功能

**memcached**

是一套分布式的快取系统，当初是Danga Interactive为了LiveJournal所发展的，但被许多软件（如MediaWiki）所使用。这是一套开放源代码软件，以BSD license授权协议发布。

memcached缺乏认证以及安全管制，这代表应该将memcached服务器放置在防火墙后。

memcached的API使用32位元的[循环冗余校验](https://baike.baidu.com/item/循环冗余校验)（CRC-32）计算键值后，将资料分散在不同的机器上。当表格满了以后，接下来新增的资料会以LRU机制替换掉。由于memcached通常只是当作快取系统使用，所以使用memcached的应用程式在写回较慢的系统时（像是后端的数据库）需要额外的程式码更新memcached内的资料

memcached 是以LiveJournal 旗下Danga Interactive 公司的Brad Fitzpatric 为首开发的一款软件。已成为mixi、hatena、Facebook、Vox、LiveJournal等众多服务中提高Web应用扩展性的重要因素。许多Web应用都将数据保存到[RDBMS](https://baike.baidu.com/item/RDBMS)中，应用服务器从中读取数据并在浏览器中显示。但随着数据量的增大、访问的集中，就会出现RDBMS的负担加重、数据库响应恶化、网站显示延迟等重大影响。这时就该memcached大显身手了。memcached是高性能的分布式内存[缓存服务器](https://baike.baidu.com/item/缓存服务器)。一般的使用目的是，通过[缓存](https://baike.baidu.com/item/缓存)数据库查询结果，减少数据库访问次数，以提高动态Web应用的速度、提高可扩展性。

Memcached 的守护进程（daemon ）是用C写的，但是客户端可以用任何语言来编写，并通过memcached协议与守护进程通信。但是它并不提供[冗余](https://baike.baidu.com/item/冗余)（例如，复制其hashmap条目）；当某个服务器S停止运行或崩溃了，所有存放在S上的键/值对都将丢失。Memcached由Danga Interactive开发，其最新版本发布于2010年，作者为Anatoly Vorobey和Brad Fitzpatrick。用于提升LiveJournal . com访问速度的。LJ每秒[动态页面](https://baike.baidu.com/item/动态页面)访问量几千次，用户700万。Memcached将数据库负载大幅度降低，更好的分配资源，更快速访问。

## 特征

* 协议简单

> memcached的服务器客户端通信并不使用复杂的XML等格式，而使用简单的基于文本行的协议。
>
> 因此，通过telnet也能在memcached上保存数据、取得数据。下面是例子。
>
> $ telnet localhost 11211
>
> Trying 127.0.0.1
>
> Connected to localhost.localdomain \(127.0.0.1\).
>
> Escape character is '^\]'.
>
> set foo 0 0 3 （保存命令）
>
> bar （数据）
>
> STORED （结果）
>
> get foo （取得命令）
>
> VALUE foo 0 3 （数据）
>
> bar （数据）
>
> memcached有两种协议，文本协议和二进制协议，文本协议以换行表示一个请求结束。请求内部参数以空格分隔。为了二进制安全，在二进制数据前要加上长度这个参数，如set x 3 abc\r\n。
>
> 文本协议固然简单，可是当请求很小的时候，过多的协议本身的数据则显得浪费（比如每个HTTP请求只发送一个字母，可HTTP头可能有上百的字符，太浪费了），而且，server在接收到请求之后还要做文本解析，也耗cpu，于是memcached又推出了二进制协议，为了锦上添花，使得memcached更加高效。二进制协议的优点就是高效，因为所有信息都以最少的数据量来表达，且server解析请求时做的是数学比较而不是字符串比较，效率高很多。
>
> memcached的协议比起HTTP来就轻得多了（当然，两者所面向的场景不一样，故这个比较没太大意义）。但是，也有缺点，就是扩展性较差，协议定得比较死，哪个位置上有哪些东西，是什么意义是定死的。所以，直接哪来用在不同的场景下不大现实。

* 基于[libevent](https://baike.baidu.com/item/libevent)的事件处理

> * libevent是个[程序库](https://baike.baidu.com/item/程序库)，它将Linux的epoll、BSD类操作系统的kqueue等事件处理功能封装成统一的接口。即使对服务器的连接数增加，也能发挥O\(1\)的性能。memcached使用这个libevent库，因此能在Linux、BSD、Solaris等操作系统上发挥其高性能。关于事件处理这里就不再详细介绍，可以参考Dan Kegel的The C10K Problem。

* 内置内存存储方式

> 为了提高性能，memcached中保存的数据都存储在memcached内置的内存存储空间中。由于数据仅存在于内存中，因此重启memcached、重启操作系统会导致全部数据消失。另外，内容容量达到指定值之后，就基于LRU\(Least Recently Used\)算法自动删除不使用的[缓存](https://baike.baidu.com/item/缓存)。memcached本身是为缓存而设计的服务器，因此并没有过多考虑数据的永久性问题。

* memcached不互相通信的分布式

> _**memcached尽管是“分布式“**_[_**缓存服务器**_](https://baike.baidu.com/item/缓存服务器)_**，但服务器端并没有分布式功能**_。各个memcached不会互相通信以共享信息。那么，怎样进行分布式呢？这完全取决于客户端的实现。本文也将介绍memcached的分布式。

## memcached分布式算法

* #### 余数计算分散法

#### 是memcached标准的memcached分布式方法，算法如下：

```
CRC($key)%N
```

> 该算法下，客户端首先根据key来计算CRC，然后结果对服务器数进行取模得到memcached服务器节点，对于这种方式有两个问题值得说明一下：
>
> 当选择到的服务器无法连接的时候，一种解决办法是将尝试的连接次数加到key后面，然后重新进行hash，这种做法也叫rehash。
>
> 第二个问题也是这种方法的致命的缺点，尽管余数计算分散发相当简单，数据分散也很优秀，当添加或者移除服务器的时候，缓存重组的代价相当大。

* #### Consistent Hashing算法（一致哈希算法）

对于memcached的分布式完全就是依靠客户端的一致哈希算法来达到分布式的存储，因为本身各个memcached的服务器之间没办法通信，并不存在副本集或者主从的概念。Consistent Hashing算法描述如下：首先求出memcached服务器节点的哈希值，并将其分配到0~2^32的圆上，这个圆我们可以把它叫做值域，然后用同样的方法求出存储数据键的哈希值，并映射到圆上。然后从数据映射到的位置开始顺时针查找，将数据保存到找到的第一个服务器上，如果超过0~2^32仍找不到，就会保存在第一台memcached服务器上，如图

![](/assets/import-memcached-01.png)

然而当添加新的memcached节点的时候必然会打乱现有这个圆的结构，这时候是没办法完全保证你以前的key依然会存在之前的节点上，但是这种结构却是能保证在添加缓存服务器的时候把损失降到最小，受结构调整后key不能命中的只有在这个圆上新增的服务器节点逆时针的第一台服务器上，其他的是不受影响的，借用图如下：

![](/assets/import-memcached-02.png)

memcached和redis一样内部的存储都是key/Value的形式，正是这种哈希表数据结构保证了在内存中查找的时间的复杂度为O\(1\)，整体上memcached的高性能这两个哈希结构起了很大的作用，当然还有memcached的多路复用I/O模型也在高并发下起到了很大的作用。另外memcached的内部操作还具有CAS原子操作，这种利用CPU指令集的操作来保证在单个节点下数据的一致性，效率相对来说比加锁要高很多。







