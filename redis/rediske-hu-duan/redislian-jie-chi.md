# redis连接池

## **简述**

合理的JedisPool资源池参数设置能为业务使用Redis保驾护航，本文将对JedisPool的使用、资源池的参数进行详细说明，最后给出“最合理”配置。

## **为什么使用连接池？**

首先Redis也是一种数据库，它基于C/S模式，因此如果需要使用必须建立连接，稍微熟悉网络的人应该都清楚地知道为什么需要建立连接，C/S模式本身就是一种远程通信的交互模式，因此Redis服务器可以单独作为一个数据库服务器来独立存在。假设Redis服务器与客户端分处在异地，虽然基于内存的Redis数据库有着超高的性能，但是底层的网络通信却占用了一次数据请求的大量时间，因为每次数据交互都需要先建立连接，假设一次数据交互总共用时30ms，超高性能的Redis数据库处理数据所花的时间可能不到1ms，也即是说前期的连接占用了29ms，连接池则可以实现在客户端建立多个链接并且不释放，当需要使用连接的时候通过一定的算法获取已经建立的连接，使用完了以后则还给连接池，这就免去了数据库连接所占用的时间。

## 使用场景

对于一些大对象，或者初始化过程较长的可复用的对象，我们如果每次都new对象出来，那么意味着会耗费大量的时间。我们可以将这些对象缓存起来，当接口调用完毕后，不是销毁对象，当下次使用的时候，直接从对象池中拿出来即可。

### 一、使用方法 {#2}

以官方的2.9.0为例子\([Jedis Release](https://github.com/xetorthio/jedis/releases)\)，Maven依赖如下：

```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
    <scope>compile</scope>
</dependency>

```

Jedis使用apache commons-pool2对Jedis资源池进行管理，所以在定义JedisPool时一个很重要的参数就是资源池GenericObjectPoolConfig，使用方式如下，其中有很多资源管理和使用的参数\(具体看第二节\)

注意：后面会提到建议用JedisPoolConfig代替GenericObjectPoolConfig

```
GenericObjectPoolConfig jedisPoolConfig = new GenericObjectPoolConfig();
jedisPoolConfig.setMaxTotal(..);
jedisPoolConfig.setMaxIdle(..);
jedisPoolConfig.setMinIdle(..);
jedisPoolConfig.setMaxWaitMillis(..);
```

JedisPool的初始化如下：

```
// redisHost和redisPort是实例的IP和端口
// redisPassword是实例的密码
// timeout，这里既是连接超时又是读写超时，从Jedis 2.8开始有区分connectionTimeout和soTimeout的构造函数

JedisPool jedisPool = new JedisPool(jedisPoolConfig, redisHost, redisPort, timeout, redisPassword);
Jedis jedis = null;
try {
    jedis = jedisPool.getResource();
    //具体的命令
    jedis.executeCommand()
} catch (Exception e) {
    logger.error(e.getMessage(), e);
} finally {
    if (jedis != null) 
        jedis.close(); //注意这里不是关闭连接，在JedisPool模式下，Jedis会被归还给资源池。
}
```

### 二、参数说明 {#3}

JedisPool保证资源在一个可控范围内，并且提供了线程安全，但是一个合理的GenericObjectPoolConfig配置能为应用使用Redis保驾护航，下面将对它的一些重要参数进行说明和建议：

在当前环境下，Jedis连接就是资源，JedisPool管理的就是Jedis连接。

#### 1. 资源设置和使用 {#4}

| 序号 | 参数名 | 含义 | 默认值 | 使用建议 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | maxTotal | 资源池中最大连接数 | 8 | 设置建议见下节 |
| 2 | maxIdle | 资源池允许最大空闲的连接数 | 8 | 设置建议见下节 |
| 3 | minIdle | 资源池确保最少空闲的连接数 | 0 | 设置建议见下节 |
| 4 | blockWhenExhausted | 当资源池用尽后，调用者是否要等待。只有当为true时，下面的maxWaitMillis才会生效 | true | 建议使用默认值 |
| 5 | maxWaitMillis | 当资源池连接用尽后，调用者的最大等待时间\(单位为毫秒\) | -1：表示永不超时 | 不建议使用默认值 |
| 6 | testOnBorrow | 向资源池借用连接时是否做连接有效性检测\(ping\)，无效连接会被移除 | false | 业务量很大时候建议设置为false\(多一次ping的开销\)。 |
| 7 | testOnReturn | 向资源池归还连接时是否做连接有效性检测\(ping\)，无效连接会被移除 | false | 业务量很大时候建议设置为false\(多一次ping的开销\)。 |
| 8 | jmxEnabled | 是否开启jmx监控，可用于监控 | true | 建议开启，但应用本身也要开启 |

#### 2.空闲资源监测 {#5}

空闲Jedis对象检测，下面四个参数组合来完成，testWhileIdle是该功能的开关。

| 序号 | 参数名 | 含义 | 默认值 | 使用建议 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | testWhileIdle | 是否开启空闲资源监测 | false | true |
| 2 | timeBetweenEvictionRunsMillis | 空闲资源的检测周期\(单位为毫秒\) | -1：不检测 | 建议设置，周期自行选择，也可以默认也可以使用下面JedisPoolConfig中的配置 |
| 3 | minEvictableIdleTimeMillis | 资源池中资源最小空闲时间\(单位为毫秒\)，达到此值后空闲资源将被移除 | 1000 60 30 = 30分钟 | 可根据自身业务决定，大部分默认值即可，也可以考虑使用下面JeidsPoolConfig中的配置 |
| 4 | numTestsPerEvictionRun | 做空闲资源检测时，每次的采样数 | 3 | 可根据自身应用连接数进行微调,如果设置为-1，就是对所有连接做空闲监测 |

为了方便使用，Jedis提供了JedisPoolConfig，它本身继承了GenericObjectPoolConfig设置了一些空闲监测设置

```
public class JedisPoolConfig extends GenericObjectPoolConfig {
  public JedisPoolConfig() {
    // defaults to make your life with connection pool easier :)
    setTestWhileIdle(true);
    //
    setMinEvictableIdleTimeMillis(60000);
    //
    setTimeBetweenEvictionRunsMillis(30000);
    setNumTestsPerEvictionRun(-1);
    }
}
```

所有默认值可以从org.apache.commons.pool2.impl.BaseObjectPoolConfig中看到。

### 三、资源池大小\(maxTotal\)、空闲\(maxIdle minIdle\)设置建议 {#6}

#### 1.maxTotal：最大连接数 {#7}

实际上这个是一个很难回答的问题，考虑的因素比较多：

* 业务希望Redis并发量
* 客户端执行命令时间
* Redis资源：例如 nodes\(例如应用个数\) \* maxTotal 是不能超过redis的最大连接数。
* 资源开销：例如虽然希望控制空闲连接，但是不希望因为连接池的频繁释放创建连接造成不必靠开销。

以一个例子说明，假设:

* 一次命令时间（borrow\|return resource + Jedis执行命令\(含网络\) ）的平均耗时约为1ms，一个连接的QPS大约是1000
* 业务期望的QPS是50000

那么理论上需要的资源池大小是50000 / 1000 = 50个。但事实上这是个理论值，还要考虑到要比理论值预留一些资源，通常来讲maxTotal可以比理论值大一些。

但这个值不是越大越好，一方面连接太多占用客户端和服务端资源，另一方面对于Redis这种高QPS的服务器，一个大命令的阻塞即使设置再大资源池仍然会无济于事。

#### 2. maxIdle minIdle {#8}

maxIdle实际上才是业务需要的最大连接数，maxTotal是为了给出余量，所以maxIdle不要设置过小，否则会有new Jedis\(新连接\)开销，而minIdle是为了控制空闲资源监测。

连接池的最佳性能是maxTotal = maxIdle ,这样就避免连接池伸缩带来的性能干扰。但是如果并发量不大或者maxTotal设置过高，会导致不必要的连接资源浪费。  
可以根据实际总OPS和调用redis客户端的规模整体评估每个节点所使用的连接池。

#### 3.监控 {#9}

实际上最靠谱的值是通过监控来得到“最佳值”的，可以考虑通过一些手段\(例如jmx\)实现监控，找到合理值。

### 四、常见问题 {#10}

#### 1.资源“不足" {#11}

```
redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool
…
Caused by: java.util.NoSuchElementException: Timeout waiting for idle object
at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:449)
```

或者

```
redis.clients.jedis.exceptions.JedisConnectionException: Could not get a resource from the pool
…
Caused by: java.util.NoSuchElementException: Pool exhausted
at org.apache.commons.pool2.impl.GenericObjectPool.borrowObject(GenericObjectPool.java:464)
```



两种情况均属于无法从资源池获取到资源，但第一种是超时，第二种是因为blockWhenExhausted为false根本就不等。

遇到此类异常，不要盲目的认为资源池不够大，第三节已经进行了分析。具体原因可以排查：网络、资源池参数设置、资源池监控\(如果对jmx监控\)、代码\(例如没执行jedis.close\(\)\)、慢查询、DNS等问题。

具体可以参考该文章：[https://www.atatech.org/articles/77799](https://www.atatech.org/articles/77799)

#### 2. 预热JedisPool {#12}

由于一些原因\(例如超时时间设置较小原因\)，有的项目在启动成功后会出现超时。JedisPool定义最大资源数、最小空闲资源数时，不会真的把Jedis连接放到池子里，第一次使用时，池子没有资源使用，会new Jedis，使用后放到池子里，可能会有一定的时间开销，所以也可以考虑在JedisPool定义后，为JedisPool提前进行预热，例如以最小空闲数量为预热数量

```
List<Jedis> minIdleJedisList = new ArrayList<Jedis>(jedisPoolConfig.getMinIdle());

for (int i = 0; i < jedisPoolConfig.getMinIdle(); i++) {
    Jedis jedis = null;
    try {
        jedis = pool.getResource();
        minIdleJedisList.add(jedis);
        jedis.ping();
    } catch (Exception e) {
        logger.error(e.getMessage(), e);
    } finally {
    }
}

for (int i = 0; i < jedisPoolConfig.getMinIdle(); i++) {
    Jedis jedis = null;
    try {
        jedis = minIdleJedisList.get(i);
        jedis.close();
    } catch (Exception e) {
        logger.error(e.getMessage(), e);
    } finally {
    
    }
}
```

## demo示例：

[https://github.com/tuonioooo](https://github.com/tuonioooo)

