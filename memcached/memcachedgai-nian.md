# Memcached概念

## 概述

Memcached 是一个高性能的分布式内存对象缓存系统，用于动态Web应用以减轻数据库负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高动态、数据库驱动网站的速度。Memcached基于一个存储键/值对的[hashmap](https://baike.baidu.com/item/hashmap/1167707)。其[守护进程](https://baike.baidu.com/item/守护进程/966835)（daemon ）是用[C](https://baike.baidu.com/item/C/7252092)写的，但是客户端可以用任何语言来编写，并通过memcached协议与守护进程通信。

## 功能

**memcached**

是一套分布式的快取系统，当初是Danga Interactive为了LiveJournal所发展的，但被许多软件（如MediaWiki）所使用。这是一套开放源代码软件，以BSD license授权协议发布。

\[1\]



memcached缺乏认证以及安全管制，这代表应该将memcached服务器放置在防火墙后。

\[1\]



memcached的API使用32位元的

[循环冗余校验](https://baike.baidu.com/item/%E5%BE%AA%E7%8E%AF%E5%86%97%E4%BD%99%E6%A0%A1%E9%AA%8C)

（CRC-32）计算键值后，将资料分散在不同的机器上。当表格满了以后，接下来新增的资料会以LRU机制替换掉。由于memcached通常只是当作快取系统使用，所以使用memcached的应用程式在写回较慢的系统时（像是后端的数据库）需要额外的程式码更新memcached内的资料

\[1\]



memcached 是以LiveJournal 旗下Danga Interactive 公司的Brad Fitzpatric 为首开发的一款软件。已成为mixi、hatena、Facebook、Vox、LiveJournal等众多服务中提高Web应用扩展性的重要因素。许多Web应用都将数据保存到

[RDBMS](https://baike.baidu.com/item/RDBMS)

中，应用服务器从中读取数据并在浏览器中显示。但随着数据量的增大、访问的集中，就会出现RDBMS的负担加重、数据库响应恶化、网站显示延迟等重大影响。

这时就该memcached大显身手了。memcached是高性能的分布式内存

[缓存服务器](https://baike.baidu.com/item/%E7%BC%93%E5%AD%98%E6%9C%8D%E5%8A%A1%E5%99%A8)

。一般的使用目的是，通过

[缓存](https://baike.baidu.com/item/%E7%BC%93%E5%AD%98)

数据库查询结果，减少数据库访问次数，以提高动态Web应用的速度、提高可扩展性。

Memcached 的守护进程（daemon ）是用C写的，但是客户端可以用任何语言来编写，并通过memcached协议与守护进程通信。但是它并不提供

[冗余](https://baike.baidu.com/item/%E5%86%97%E4%BD%99)

（例如，复制其hashmap条目）；当某个服务器S停止运行或崩溃了，所有存放在S上的键/值对都将丢失。

Memcached由Danga Interactive开发，其最新版本发布于2010年，作者为Anatoly Vorobey和Brad Fitzpatrick。用于提升LiveJournal . com访问速度的。LJ每秒

[动态页面](https://baike.baidu.com/item/%E5%8A%A8%E6%80%81%E9%A1%B5%E9%9D%A2)

访问量几千次，用户700万。Memcached将数据库负载大幅度降低，更好的分配资源，更快速访问。







  


