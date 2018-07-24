# 实现定时消息通知

## 一、需求分析：

1. 设置了生存时间的Key，在过期时能不能有所提示？
2. 如果能对过期Key有个监听，如何对过期Key进行一个回调处理？
3. 如何使用 Redis 来实现定时任务？

## 二、序言：

本文所说的定时任务或者说计划任务并不是很多人想象中的那样，比如说每天凌晨三点自动运行起来跑一个脚本。这种都已经烂大街了，随便一个 Crontab 就能搞定了。

这里所说的定时任务可以说是计时器任务，比如说用户触发了某个动作，那么从这个点开始过二十四小时我们要对这个动作做点什么。那么如果有 1000 个用户触发了这个动作，就会有 1000 个定时任务。于是这就不是 Cron 范畴里面的内容了。

举个最简单的例子，一个用户推荐了另一个用户，我们定一个二十四小时之后的任务，看看被推荐的用户有没有来注册，如果没注册就给他搞一条短信过去。

## 三、Redis介绍

在 Redis 的 2.8.0 版本之后，其推出了一个新的特性——键空间消息（Redis Keyspace Notifications），它配合 2.0.0 版本之后的 SUBSCRIBE 就能完成这个定时任务的操作了，不过定时的单位是秒。

（1）Publish / Subscribe

Redis 在 2.0.0 之后推出了 Pub / Sub 的指令，大致就是说一边给 Redis 的特定频道发送消息，另一边从 Redis 的特定频道取值——形成了一个简易的消息队列。

（2）Redis Keyspace Notifications

在 Redis 里面有一些事件，比如键到期、键被删除等。然后我们可以通过配置一些东西来让 Redis 一旦触发这些事件的时候就往特定的 Channel 推一条消息。

大致的流程就是我们给 Redis 的某一个 db 设置过期事件，使其键一旦过期就会往特定频道推消息，我在自己的客户端这边就一直消费这个频道就好了。

以后一来一条定时任务，我们就把这个任务状态压缩成一个键，并且过期时间为距这个任务执行的时间差。那么当键一旦到期，就到了任务该执行的时间，Redis 自然会把过期消息推去，我们的客户端就能接收到了。这样一来就起到了定时任务的作用。

## 四、Key过期事件的Redis配置

这里需要配置 notify-keyspace-events 的参数为 “Ex”。x 代表了过期事件。notify-keyspace-events "Ex" 保存配置后，重启Redis服务，使配置生效。  
重启Reids服务器：

```
root@iZ23s8agtagZ:/etc/redis# service redis-server restart redis.conf
Stopping redis-server: redis-server.
Starting redis-server: redis-server.
```

添加过期事件订阅 开启一个终端，redis-cli 进入 redis 。开始订阅所有操作，等待接收消息。

```
tinywan@iZ23a7607jaZ:~$ redis-cli -h 127.0.01.4 -p 63789
127.0.0.1:63789> psubscribe __keyevent@0__:expired
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "__keyevent@0__:expired"
3) (integer) 1
```

再开启一个终端，redis-cli 进入 redis，新增一个 20秒过期的键：

```
1270.01.1.1:63789> SETEX coolName 123 20
OK
121.41.188.109:63789> get coolName
"20"
121.41.188.109:63789> ttl coolName
(integer) 104
```

> setex 用法 见官方文档：[http://doc.redisfans.com/string/setex.html](http://doc.redisfans.com/string/setex.html)

另外一边执行了阻塞订阅操作后的终端，20秒过期后有如下信息输出：

```
121.141.188.209:63789> psubscribe __keyevent@0__:expired
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "__keyevent@0__:expired"
3) (integer) 1
1) "pmessage"
2) "__keyevent@0__:expired"
3) "__keyevent@0__:expired"
4) "coolName"
```

说明：说明对过期Key信息的订阅是成功的。

## 五、Jedis实现Publish/Subscribe功能

Redis为我们提供了publish/subscribe\(发布/订阅\)功能。我们可以对某个channel\(频道\)进行subscribe\(订阅\)，当有人在这个channel上publish\(发布\)消息时，redis就会通知我们，这样我们可以收到别人发布的消息。  
作为Java的redis客户端，Jedis提供了publish/subscribe的接口。本文讲述如何使用Jedis来实现redis的publish/subscribe。

* ### 定义Subscriber类

Jedis定义了抽象类`JedisPubSub`，在这个类中，定义publish/subsribe的回调方法。通过继承`JedisPubSub`类并重新实现这些回调方法，当publish/subsribe事件发生时，我们可以定制自己的处理逻辑。

在以下例子中，我们定义了`Subscriber`类，这个类继承了`JedisPubSub`类，并重新实现了其中的回调方法。

_Subscriber.java_

```
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;


public class SubThread extends Thread {
    private final JedisPool jedisPool;
    private final Subscriber subscriber = new Subscriber();

    private final String channel = "mychannel";

    public SubThread(JedisPool jedisPool) {
        super("SubThread");
        this.jedisPool = jedisPool;
    }

    @Override
    public void run() {
        System.out.println(String.format("subscribe redis, channel %s, thread will be blocked", channel));
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            jedis.subscribe(subscriber, channel);
        } catch (Exception e) {
            System.out.println(String.format("subsrcibe channel error, %s", e));
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
    }
}
```

* ### 定义SubThread线程类

由于Jedis的`subscribe`操作是阻塞的，因此，我们另起一个线程来进行subscribe操作。

_SubThread.java_

```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

public class Publisher {
    private final JedisPool jedisPool;

    public Publisher(JedisPool jedisPool) {
        this.jedisPool = jedisPool;
    }

    public void start() {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        Jedis jedis = jedisPool.getResource();
        while (true) {
            String line = null;
            try {
                line = reader.readLine();
                if (!"quit".equals(line)) {
                    jedis.publish("mychannel", line);
                } else {
                    break;
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
         }
    }
}
```

在上面的代码中，我们从`JedisPool`获取一个`Jedis`实例，并使用这个`Jedis`实例进行`subscribe`的操作。  
`Jedis`的`subscribe`的声明如下：

> public void subscribe\(final JedisPubSub jedisPubSub, final String… channels\)

第一个参数接受一个`JedisPubSub`对象，第二个参数指定对哪个频道进行订阅。上例中，我们把我们定义的`Subscriber`对象传给`subscribe`方法。  
当publish/subscribe的事件发生时，会自动调用我们`Subscriber`的方法。

* ### 定义Publisher类

`Publisher`类接受用户的输入，并将输入发布到channel。当用户输入”quit”后，输入结束。

_Publisher.java_

```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;

public class Publisher {
    private final JedisPool jedisPool;

    public Publisher(JedisPool jedisPool) {
        this.jedisPool = jedisPool;
    }

    public void start() {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        Jedis jedis = jedisPool.getResource();
        while (true) {
            String line = null;
            try {
                line = reader.readLine();
                if (!"quit".equals(line)) {
                    jedis.publish("mychannel", line);
                } else {
                    break;
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
         }
    }
}
```

# 定义入口代码 {#定义入口代码}

如下是我们的程序入口代码。

_PubSubDemo.java_

```
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;


public class PubSubDemo 
{
    public static void main( String[] args )
    {
        // 替换成你的reids地址和端口
        String redisIp = "192.168.229.154";
        int reidsPort = 6379;
        JedisPool jedisPool = new JedisPool(new JedisPoolConfig(), redisIp, reidsPort);
        System.out.println(String.format("redis pool is starting, redis ip %s, redis port %d", redisIp, reidsPort));

        SubThread subThread = new SubThread(jedisPool);
        subThread.start();

        Publisher publisher = new Publisher(jedisPool);
        publisher.start();
    }
}
```

在上面的代码中，我们首先生成了一个`JedisPool`的redis连接池，这是由于`Jedis`不是线程安全的，`JedisPool`是线程安全的。而我们的程序在主线程和订阅线程\(SubThread\)均需要使用`Jedis`，故在程序中我们使用`JedisPool`。具体也可以参考[在多线程环境中使用Jedis](http://blog.csdn.net/lihao21/article/details/46830553)。  
由于`Jedis`的subcribe操作是阻塞的，故我们另起了一个线程来进行subcribe操作。  
通过调用`Publisher::start()`方法，接受用户的输入，并publish到指定的channel。

# 输出 {#输出}

> redis pool is starting, redis ip 192.168.229.154, redis port 6379  
> subscribe redis, channel mychannel, thread will be blocked  
> subscribe redis channel success, channel mychannel, subscribedChannels 1

这时输入

> hello

控制窗口中输出

> receive redis published message, channel mychannel, message hello

# 参考资料 {#参考资料}

* [https://github.com/xetorthio/jedis/wiki/AdvancedUsage](https://github.com/xetorthio/jedis/wiki/AdvancedUsage)
* [http://basrikahveci.com/a-simple-jedis-publish-subscribe-example/](http://basrikahveci.com/a-simple-jedis-publish-subscribe-example/)
* [https://blog.csdn.net/canot/article/details/51938955](https://blog.csdn.net/canot/article/details/51938955)       java的实现方式



