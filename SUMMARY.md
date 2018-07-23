# Summary

* [Introduction](README.md)
* [Redis](redis.md)
  * [redis常用命令](redis/redischang-yong-ming-ling.md)
    * [生产环境谨慎使用keys命令](redis/redischang-yong-ming-ling/sheng-chan-huan-jing-jin-shen-shi-yong-keys-ming-ling.md)
    * [Redis 命令参考](redis/redischang-yong-ming-ling/redisming-ling.md)
  * [redis客户端](redis/rediske-hu-duan.md)
    * [Jedis使用指南](redis/rediske-hu-duan/jedis.md)
    * [redisson](redis/rediske-hu-duan/redisson.md)
    * [redis连接池](redis/rediske-hu-duan/redislian-jie-chi.md)
    * [jedisCluster详解](redis/rediske-hu-duan/jediscluster.md)
    * [jedisCluster+SpringMVC整合](redis/rediske-hu-duan/jediscluster+springmvczheng-he.md)
  * [redis主从模式](redis/rediszhu-cong-mo-shi.md)
    * [配置方式](redis/rediszhu-cong-mo-shi/pei-zhi-fang-shi.md)
    * [一主多从读写分离架构](redis/rediszhu-cong-mo-shi/yi-zhu-duo-cong.md)
  * [redis持计划机制的讲解](redis/redischi-ji-hua-ji-zhi-de-jiang-jie.md)
    * [持久化](redis/redischi-ji-hua-ji-zhi-de-jiang-jie/chi-jiu-hua.md)
    * [RDB详解](redis/redischi-ji-hua-ji-zhi-de-jiang-jie/rdbxiang-jie.md)
    * [AOF详解](redis/redischi-ji-hua-ji-zhi-de-jiang-jie/aofxiang-jie.md)
    * [RDB与AOF异同比较](redis/redischi-ji-hua-ji-zhi-de-jiang-jie/yi-tong-bi-jiao.md)
  * [哨兵机制详解（sentinel）](redis/shao-bing-ji-zhi-xiang-jie.md)
    * [哨兵使用架构（sentinel）](redis/shao-bing-ji-zhi-xiang-jie/shao-bing-shi-yong-jia-gou.md)
    * [哨兵如何监控redis](redis/shao-bing-ji-zhi-xiang-jie/shao-bing-ru-he-jian-kong-redis.md)
    * 哨兵如何高可用
    * 哨兵内部通讯原理
    * 基于哨兵的redis高可用架构
  * [redisCluster集群](redis/redisclusterji-qun.md)
    * 架构分析
    * 搭建高可用集群
    * 分区存储详解
    * [java客户端使用redisCluster](redis/redisclusterji-qun/javake-hu-duan-shi-yong-rediscluster.md)
  * [实战场景](redis/shi-zhan-chang-jing.md)
    * 高性能分布式缓存
    * 实现定时消息通知
    * 简单高速队列的实现与应用
    * 实现去重幂等性
    * 数据计数/订单号生成
    * 基于redis实现分布式锁
* [Memcached](memcached.md)
  * [Memcached概念](memcached/memcachedgai-nian.md)
  * [安装配置](memcached/an-zhuang-pei-zhi.md)
    * [Linux安装](memcached/an-zhuang-pei-zhi/linuxan-zhuang.md)
    * [Windows安装](memcached/an-zhuang-pei-zhi/windowsan-zhuang.md)
  * [集群搭建](memcached/ji-qun-da-jian.md)
  * [常用场景](memcached/chang-yong-chang-jing.md)
  * [客户端命令](memcached/ke-hu-duan-ming-ling.md)
    * [客户端连接命令](memcached/ke-hu-duan-ming-ling/ke-hu-duan-lian-jie-ming-ling.md)
    * [Memcached 存储命令](memcached/ke-hu-duan-ming-ling/memcached-cun-chu-ming-ling.md)
      * [Memcached set 命令](memcached/ke-hu-duan-ming-ling/memcached-cun-chu-ming-ling/memcached-set-ming-ling.md)
      * [Memcached add 命令](memcached/ke-hu-duan-ming-ling/memcached-cun-chu-ming-ling/memcached-add-ming-ling.md)
      * [Memcached replace 命令](memcached/ke-hu-duan-ming-ling/memcached-cun-chu-ming-ling/memcached-replace-ming-ling.md)
      * [Memcached append 命令](memcached/ke-hu-duan-ming-ling/memcached-cun-chu-ming-ling/memcached-append-ming-ling.md)
      * [Memcached prepend 命令](memcached/ke-hu-duan-ming-ling/memcached-cun-chu-ming-ling/memcached-prepend-ming-ling.md)
      * [Memcached CAS 命令](memcached/ke-hu-duan-ming-ling/memcached-cun-chu-ming-ling/memcached-cas-ming-ling.md)
    * [Memcached 查找命令](memcached/ke-hu-duan-ming-ling/memcached-cha-zhao-ming-ling.md)
      * [Memcached get 命令](memcached/ke-hu-duan-ming-ling/memcached-cha-zhao-ming-ling/memcached-get-ming-ling.md)
      * [Memcached gets 命令](memcached/ke-hu-duan-ming-ling/memcached-cha-zhao-ming-ling/memcached-gets-ming-ling.md)
      * [Memcached delete 命令](memcached/ke-hu-duan-ming-ling/memcached-cha-zhao-ming-ling/memcached-delete-ming-ling.md)
      * [Memcached incr 与 decr 命令](memcached/ke-hu-duan-ming-ling/memcached-cha-zhao-ming-ling/memcached-incr-yu-decr-ming-ling.md)
    * [Memcached 统计命令](memcached/ke-hu-duan-ming-ling/memcached-tong-ji-ming-ling.md)
      * [Memcached stats 命令](memcached/ke-hu-duan-ming-ling/memcached-tong-ji-ming-ling/memcached-stats-ming-ling.md)
      * [Memcached stats items 命令](memcached/ke-hu-duan-ming-ling/memcached-tong-ji-ming-ling/memcached-stats-items-ming-ling.md)
      * [Memcached stats slabs 命令](memcached/ke-hu-duan-ming-ling/memcached-tong-ji-ming-ling/memcached-stats-slabs-ming-ling.md)
      * [Memcached stats sizes 命令](memcached/ke-hu-duan-ming-ling/memcached-tong-ji-ming-ling/memcached-stats-sizes-ming-ling.md)
      * [Memcached flush\_all 命令](memcached/ke-hu-duan-ming-ling/memcached-tong-ji-ming-ling/memcached-flushallming-ling.md)
  * [Java客户端](memcached/javake-hu-duan.md)
  * [memcached特点与redis区别](memcached/memcachedte-dian-yu-redis-qu-bie.md)
  * [一致性哈希算法原理](memcached/yi-zhi-xing-ha-xi-suan-fa-yuan-li.md)
  * [Memcache 高可用集群之magent实现主从](memcached/magentshi-xian-memcache-zhu-cong.md)
* [互联网缓存架构设计](hu-lian-wang-huan-cun-jia-gou-she-ji.md)
  * 常见的缓存架构方案
  * 缓存雪崩的解决方案
  * 缓存穿透的解决方案

