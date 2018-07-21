# Memcached概念

## 概述

Memcached 是一个高性能的分布式内存对象缓存系统，用于动态Web应用以减轻数据库负载。它通过在内存中缓存数据和对象来减少读取数据库的次数，从而提高动态、数据库驱动网站的速度。Memcached基于一个存储键/值对的[hashmap](https://baike.baidu.com/item/hashmap/1167707)。其[守护进程](https://baike.baidu.com/item/%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B/966835)（daemon ）是用[C](https://baike.baidu.com/item/C/7252092)写的，但是客户端可以用任何语言来编写，并通过memcached协议与守护进程通信。





