# Jedis使用指南

## 概述

Jedis Client是Redis官网推荐的一个面向java客户端，库文件实现了对各类API进行封装调用。

Jedis源码工程地址：

[https://github.com/xetorthio/jedis](https://github.com/xetorthio/jedis)

## 使用

* POM.xml 添加依赖

```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
    <type>jar</type>
    <scope>compile</scope>
</dependency>

```

* **示例1**

```
//连接redis ，redis的默认端口是6379

Jedis  jedis = new Jedis ("localhost",6379); 


//验证密码，如果没有设置密码这段代码省略

jedis.auth("password"); 


jedis.connect();//连接

jedis.disconnect();//断开连接


Set<String> keys = jedis.keys("*"); //列出所有的key

Set<String> keys = jedis.keys("key"); //查找特定的key


//移除给定的一个或多个key,如果key不存在,则忽略该命令. 

jedis.del("key1");

jedis.del("key1","key2","key3","key4","key5");


//移除给定key的生存时间(设置这个key永不过期)
jedis.persist("key1"); 

//检查给定key是否存在
jedis.exists("key1"); 

//将key改名为newkey,当key和newkey相同或者key不存在时,返回一个错误
jedis.rename("key1", "key2");

//返回key所储存的值的类型。 
//none(key不存在),string(字符串),list(列表),set(集合),zset(有序集),hash(哈希表) 
jedis.type("key1");

//设置key生存时间，当key过期时，它会被自动删除。 
jedis.expire("key1", 5);//5秒过期 
 


//字符串值value关联到key。 
jedis.set("key1", "value1"); 

//将值value关联到key，并将key的生存时间设为seconds(秒)。 
jedis.setex("foo", 5, "haha"); 

//清空所有的key
jedis.flushAll();

//返回key的个数 
jedis.dbSize();

//哈希表key中的域field的值设为value。 
jedis.hset("key1", "field1", "field1-value"); 
jedis.hset("key1", "field2", "field2-value"); 

Map map = new HashMap(); 
map.put("field1", "field1-value"); 
map.put("field2", "field2-value"); 
jedis.hmset("key1", map); 


//返回哈希表key中给定域field的值 
jedis.hget("key1", "field1");

//返回哈希表key中给定域field的值(多个)
List list = jedis.hmget("key1","field1","field2"); 
for(int i=0;i<list.size();i++){ 
   System.out.println(list.get(i)); 
} 

//返回哈希表key中所有域和值
Map<String,String> map = jedis.hgetAll("key1"); 
for(Map.Entry entry: map.entrySet()) { 
   System.out.print(entry.getKey() + ":" + entry.getValue() + "\t"); 
} 

//删除哈希表key中的一个或多个指定域
jedis.hdel("key1", "field1");
jedis.hdel("key1", "field1","field2");

//查看哈希表key中，给定域field是否存在。 
jedis.hexists("key1", "field1");

//返回哈希表key中的所有域
jedis.hkeys("key1");

//返回哈希表key中的所有值
jedis.hvals("key1");



//将值value插入到列表key的表头。 
jedis.lpush("key1", "value1-0"); 
jedis.lpush("key1", "value1-1"); 
jedis.lpush("key1", "value1-2"); 

//返回列表key中指定区间内的元素,区间以偏移量start和stop指定.
//下标(index)参数start和stop从0开始;
//负数下标代表从后开始(-1表示列表的最后一个元素,-2表示列表的倒数第二个元素,以此类推)
List list = jedis.lrange("key1", 0, -1);//stop下标也在取值范围内(闭区间)
for(int i=0;i<list.size();i++){ 
   System.out.println(list.get(i)); 
} 

//返回列表key的长度。 
jedis.llen("key1")



//将member元素加入到集合key当中。 
jedis.sadd("key1", "value0"); 
jedis.sadd("key1", "value1"); 

//移除集合中的member元素。 
jedis.srem("key1", "value1"); 

//返回集合key中的所有成员。 
Set set = jedis.smembers("key1"); 

//判断元素是否是集合key的成员
jedis.sismember("key1", "value2")); 

//返回集合key的元素的数量
jedis.scard("key1");
 
//返回一个集合的全部成员，该集合是所有给定集合的交集
jedis.sinter("key1","key2")
 
//返回一个集合的全部成员，该集合是所有给定集合的并集
jedis.sunion("key1","key2")

//返回一个集合的全部成员，该集合是所有给定集合的差集
jedis.sdiff("key1","key2");
```

* **示例2**

```
public void setup() {
 //连接redis服务器
        jedis = new Jedis("172.24.4.183", 6379);
//        jedis.auth("redis");//验证密码,如果需要验证的话
    }
/**
 * 键操作
 */
public void testKey()  throws InterruptedException{
    System.out.println("清空数据："+jedis.flushDB());
    System.out.println("判断某个键是否存在："+jedis.exists("username"));
    System.out.println("新增<'username','xmr'>的键值对："+jedis.set("username", "xmr"));
    System.out.println(jedis.exists("username"));
    System.out.println("新增<'password','password'>的键值对："+jedis.set("password", "123"));
    System.out.print("系统中所有的键如下：");
    Set<String> keys = jedis.keys("*");
    System.out.println(keys);
    System.out.println("删除键password:"+jedis.del("password"));
    System.out.println("判断键password是否存在："+jedis.exists("password"));
    System.out.println("设置键username的过期时间为5s:"+jedis.expire("username", 8));
    TimeUnit.SECONDS.sleep(2);
    System.out.println("查看键username的剩余生存时间："+jedis.ttl("username"));
    System.out.println("移除键username的生存时间："+jedis.persist("username"));
    System.out.println("查看键username的剩余生存时间："+jedis.ttl("username"));
    System.out.println("查看键username所存储的值的类型："+jedis.type("username"));
}

```

![](/assets/import-redis-05.png)

* **示例3  字符串操作**

```
public void testString() throws InterruptedException {
    jedis.flushDB();
    System.out.println("===========增加数据===========");
    System.out.println(jedis.set("key1", "value1"));
    System.out.println(jedis.set("key2", "value2"));
    System.out.println(jedis.set("key3", "value3"));
    System.out.println("删除键key2:" + jedis.del("key2"));
    System.out.println("获取键key2:" + jedis.get("key2"));
    System.out.println("修改key1:" + jedis.set("key1", "1"));
    System.out.println("获取key1的值：" + jedis.get("key1"));
    System.out.println("在key3后面加入值：" + jedis.append("key3", "End"));
    System.out.println("key3的值：" + jedis.get("key3"));
    System.out.println("增加多个键值对：" + jedis.mset("key01", "value01", "key02", "value02", "key03", "value03"));
    System.out.println("获取多个键值对：" + jedis.mget("key01", "key02", "key03"));
    System.out.println("获取多个键值对：" + jedis.mget("key01", "key02", "key03", "key04"));
    System.out.println("删除多个键值对：" + jedis.del(new String[]{"key01", "key02"}));
    System.out.println("获取多个键值对：" + jedis.mget("key01", "key02", "key03"));
 
    jedis.flushDB();
    System.out.println("===========新增键值对,防止覆盖原先值==============");
    System.out.println(jedis.setnx("key1", "value1"));
    System.out.println(jedis.setnx("key2", "value2"));
    System.out.println(jedis.setnx("key2", "value2-new"));
    System.out.println(jedis.get("key1"));
    System.out.println(jedis.get("key2"));
 
    System.out.println("===========新增键值对并设置有效时间=============");
    System.out.println(jedis.setex("key3", 2, "value3"));
    System.out.println(jedis.get("key3"));
    TimeUnit.SECONDS.sleep(3);
    System.out.println(jedis.get("key3"));
 
    System.out.println("===========获取原值，更新为新值==========");//GETSET is an atomic set this value and return the old value command.
    System.out.println(jedis.getSet("key2", "key2GetSet"));
    System.out.println(jedis.get("key2"));
 
    System.out.println("获得key2的值的字串：" + jedis.getrange("key2", 2, 4));
}

```

![](/assets/import-redis-06.png)

* **示例4哈希操作**

```
/**
 * redis操作Hash
 */
public void testHash() {
    jedis.flushDB();
    Map<String, String> map = new HashMap<>();
    map.put("key1", "value1");
    map.put("key2", "value2");
    map.put("key3", "value3");
    map.put("key4", "value4");
    jedis.hmset("hash", map);
    jedis.hset("hash", "key5", "value5");
    System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));//return Map<String,String>
    System.out.println("散列hash的所有键为：" + jedis.hkeys("hash"));//return Set<String>
    System.out.println("散列hash的所有值为：" + jedis.hvals("hash"));//return List<String>
    System.out.println("将key6保存的值加上一个整数，如果key6不存在则添加key6：" + jedis.hincrBy("hash", "key6", 6));
    System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));
    System.out.println("将key6保存的值加上一个整数，如果key6不存在则添加key6：" + jedis.hincrBy("hash", "key6", 3));
    System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));
    System.out.println("删除一个或者多个键值对：" + jedis.hdel("hash", "key2"));
    System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));
    System.out.println("散列hash中键值对的个数：" + jedis.hlen("hash"));
    System.out.println("判断hash中是否存在key2：" + jedis.hexists("hash", "key2"));
    System.out.println("判断hash中是否存在key3：" + jedis.hexists("hash", "key3"));
    System.out.println("获取hash中的值：" + jedis.hmget("hash", "key3"));
    System.out.println("获取hash中的值：" + jedis.hmget("hash", "key3", "key4"));
}

```

![](/assets/import-redis-07.png)

* **示例5List操作**

```
 public void testList() {
        jedis.flushDB();
        System.out.println("===========添加一个list===========");
        jedis.lpush("lists", "ArrayList", "Vector", "Stack", "HashMap", "WeakHashMap", "LinkedHashMap");
        jedis.lpush("lists", "HashSet");
        jedis.lpush("lists", "TreeSet");
        jedis.lpush("lists", "TreeMap");
        System.out.println("lists的内容：" + jedis.lrange("lists", 0, -1));//-1代表倒数第一个元素，-2代表倒数第二个元素
        System.out.println("lists区间0-3的元素：" + jedis.lrange("lists", 0, 3));
        System.out.println("===============================");
        // 删除列表指定的值 ，第二个参数为删除的个数（有重复时），后add进去的值先被删，类似于出栈
        System.out.println("删除指定元素个数：" + jedis.lrem("lists", 2, "HashMap"));
        System.out.println("lists的内容：" + jedis.lrange("lists", 0, -1));
        System.out.println("删除下表0-3区间之外的元素：" + jedis.ltrim("lists", 0, 3));
        System.out.println("lists的内容：" + jedis.lrange("lists", 0, -1));
        System.out.println("lists列表出栈（左端）：" + jedis.lpop("lists"));
        System.out.println("lists的内容：" + jedis.lrange("lists", 0, -1));
        System.out.println("lists添加元素，从列表右端，与lpush相对应：" + jedis.rpush("lists", "EnumMap"));
        System.out.println("lists的内容：" + jedis.lrange("lists", 0, -1));
        System.out.println("lists列表出栈（右端）：" + jedis.rpop("lists"));
        System.out.println("lists的内容：" + jedis.lrange("lists", 0, -1));
        System.out.println("修改lists指定下标1的内容：" + jedis.lset("lists", 1, "LinkedArrayList"));
        System.out.println("lists的内容：" + jedis.lrange("lists", 0, -1));
        System.out.println("===============================");
        System.out.println("lists的长度：" + jedis.llen("lists"));
        System.out.println("获取lists下标为2的元素：" + jedis.lindex("lists", 2));
        System.out.println("===============================");
        jedis.lpush("sortedList", "3", "6", "2", "0", "7", "4");
        System.out.println("sortedList排序前：" + jedis.lrange("sortedList", 0, -1));
        System.out.println(jedis.sort("sortedList"));
        System.out.println("sortedList排序后：" + jedis.lrange("sortedList", 0, -1));
    

```

![](/assets/import-redis-08.png)

* **集合（Set）操作**

```
public void testSet() {
jedis.flushDB();
System.out.println("============向集合中添加元素============");
System.out.println(jedis.sadd("eleSet", "e1", "e2", "e4", "e3", "e0", "e8", "e7", "e5"));
System.out.println(jedis.sadd("eleSet", "e6"));
System.out.println("eleSet的所有元素为：" + jedis.smembers("eleSet"));
System.out.println("删除一个元素e0：" + jedis.srem("eleSet", "e0"));
System.out.println("eleSet的所有元素为：" + jedis.smembers("eleSet"));
System.out.println("删除两个元素e7和e6：" + jedis.srem("eleSet", "e7", "e6"));
System.out.println("eleSet的所有元素为：" + jedis.smembers("eleSet"));
System.out.println("随机的移除集合中的一个元素：" + jedis.spop("eleSet"));
System.out.println("eleSet的所有元素为：" + jedis.smembers("eleSet"));
System.out.println("eleSet中包含元素的个数：" + jedis.scard("eleSet"));
System.out.println("e1是否在eleSet中：" + jedis.sismember("eleSet", "e1"));
System.out.println("=================================");
System.out.println(jedis.sadd("eleSet1", "e1", "e2", "e4", "e3", "e0", "e8", "e7", "e5"));
System.out.println(jedis.sadd("eleSet2", "e1", "e2", "e4", "e3", "e0", "e8"));
System.out.println("将eleSet1中删除e1并存入eleSet3中：" + jedis.smove("eleSet1", "eleSet3", "e1"));
System.out.println("eleSet1中的元素：" + jedis.smembers("eleSet1"));
System.out.println("eleSet3中的元素：" + jedis.smembers("eleSet3"));
System.out.println("============集合运算=================");
System.out.println("eleSet1中的元素：" + jedis.smembers("eleSet1"));
System.out.println("eleSet2中的元素：" + jedis.smembers("eleSet2"));
System.out.println("eleSet1和eleSet2的交集:" + jedis.sinter("eleSet1", "eleSet2"));
System.out.println("eleSet1和eleSet2的并集:" + jedis.sunion("eleSet1", "eleSet2"));
System.out.println("eleSet1和eleSet2的差集:" + jedis.sdiff("eleSet1", "eleSet2"));//eleSet1中有，eleSet2中没有
}


输出结果：
============向集合中添加元素============
8
1
eleSet的所有元素为：[e0, e5, e3, e8, e7, e2, e1, e4, e6]
删除一个元素e0：1
eleSet的所有元素为：[e5, e3, e8, e7, e2, e1, e4, e6]
删除两个元素e7和e6：2
eleSet的所有元素为：[e1, e4, e3, e5, e2, e8]
随机的移除集合中的一个元素：e1
eleSet的所有元素为：[e3, e5, e2, e8, e4]
eleSet中包含元素的个数：5
e1是否在eleSet中：false
=================================
8
6
将eleSet1中删除e1并存入eleSet3中：1
eleSet1中的元素：[e0, e5, e3, e8, e7, e2, e4]
eleSet3中的元素：[e1]
============集合运算=================
eleSet1中的元素：[e0, e5, e3, e8, e7, e2, e4]
eleSet2中的元素：[e3, e1, e4, e0, e8, e2]
eleSet1和eleSet2的交集:[e3, e4, e0, e8, e2]
eleSet1和eleSet2的并集:[e2, e1, e4, e0, e3, e5, e7, e8]
eleSet1和eleSet2的差集:[e5, e7]
```

* **有序集合**

```
public void testSortedSet()
{
jedis.flushDB();
Map<String,Double> map = new HashMap<>();
map.put("key2",1.5);
map.put("key3",1.6);
map.put("key4",1.9);
System.out.println(jedis.zadd("zset", 3,"key1"));
System.out.println(jedis.zadd("zset",map));
System.out.println("zset中的所有元素："+jedis.zrangeByScore("zset", 0,100));
System.out.println("zset中key2的分值："+jedis.zscore("zset", "key2"));
System.out.println("zset中key2的排名："+jedis.zrank("zset", "key2"));
System.out.println("删除zset中的元素key3："+jedis.zrem("zset", "key3"));
System.out.println("zset中的所有元素："+jedis.zrange("zset", 0, -1));
System.out.println("zset中元素的个数："+jedis.zcard("zset"));
System.out.println("zset中分值在1-4之间的元素的个数："+jedis.zcount("zset", 1, 4));
System.out.println("key2的分值加上5："+jedis.zincrby("zset", 5, "key2"));
System.out.println("key3的分值加上4："+jedis.zincrby("zset", 4, "key3"));
System.out.println("zset中的所有元素："+jedis.zrange("zset", 0, -1));
}
输出结果：
1
3
zset中的所有元素：[key2, key3, key4, key1]
zset中key2的分值：1.5
zset中key2的排名：0
删除zset中的元素key3：1
zset中的所有元素：[key2, key4, key1]
zset中元素的个数：3
zset中分值在1-4之间的元素的个数：3
key2的分值加上5：6.5
key3的分值加上4：4.0
zset中的所有元素：[key4, key1, key3, key2]
```

* **排序sort**

```
public void testSort()
{
jedis.flushDB();
jedis.lpush("collections", "ArrayList", "Vector", "Stack", "HashMap", "WeakHashMap", "LinkedHashMap");
System.out.println("collections的内容："+jedis.lrange("collections", 0, -1));
SortingParams sortingParameters = new SortingParams();
System.out.println(jedis.sort("collections",sortingParameters.alpha()));
System.out.println("===============================");
jedis.lpush("sortedList", "3","6","2","0","7","4");
System.out.println("sortedList排序前："+jedis.lrange("sortedList", 0, -1));
System.out.println("升序："+jedis.sort("sortedList", sortingParameters.asc()));
System.out.println("升序："+jedis.sort("sortedList", sortingParameters.desc()));
System.out.println("===============================");
jedis.lpush("userlist", "33");
jedis.lpush("userlist", "22");
jedis.lpush("userlist", "55");
jedis.lpush("userlist", "11");
jedis.hset("user:66", "name", "66");
jedis.hset("user:55", "name", "55");
jedis.hset("user:33", "name", "33");
jedis.hset("user:22", "name", "79");
jedis.hset("user:11", "name", "24");
jedis.hset("user:11", "add", "beijing");
jedis.hset("user:22", "add", "shanghai");
jedis.hset("user:33", "add", "guangzhou");
jedis.hset("user:55", "add", "chongqing");
jedis.hset("user:66", "add", "xi'an");
sortingParameters = new SortingParams();
sortingParameters.get("user:*->name");
sortingParameters.get("user:*->add");
System.out.println(jedis.sort("userlist",sortingParameters));
}
输出结果：
collections的内容：[LinkedHashMap, WeakHashMap, HashMap, Stack, Vector, ArrayList]
[ArrayList, HashMap, LinkedHashMap, Stack, Vector, WeakHashMap]
===============================
sortedList排序前：[4, 7, 0, 2, 6, 3]
升序：[0, 2, 3, 4, 6, 7]
升序：[7, 6, 4, 3, 2, 0]
===============================
[24, beijing, 79, shanghai, 33, guangzhou, 55, chongqing]
```

## JedisPool应用

虽然我们可以简单地创建Jedis使用，但每次操作的时候，都建立连接，很耗费性能。解决方法就是从一个连接池中取出连接对象，用完还回去。使用连接池的方案还能解决很多同步性问题。在Jedis中，管理Redis连接的类是JedisPool。要想使用JedisPool需要添加jar包或依赖库，在pom.xml中添加

```
<dependency>
<groupId>org.apache.commons</groupId>
<artifactId>commons-pool2</artifactId>
<version>2.4.2</version>
</dependency>
```

实现方式有两种，一种是通过配置文件

\(properties文件\)，我的文件名是jedisPool.properties：

\#最大分配的对象数

redis.pool.maxTotal=1024

\#最大能够保持idel状态的对象数

redis.pool.maxIdle=200

\#当池内没有返回对象时，最大等待时间

redis.pool.maxWait=1000

\#当调用borrow Object方法时，是否进行有效性检查

redis.pool.testOnBorrow=true

\#当调用return Object方法时，是否进行有效性检查

redis.pool.testOnReturn=true

\#IP

redis.ip=172.24.4.183

\#Port

redis.port=6379

java类如下：

public class JedisUtil {

private static Jedis jedis;

private static JedisPool jedisPool = null;

/\*\*

\* 初始化Redis连接池

\*/

static {

ResourceBundle bundle = ResourceBundle.getBundle\("jedisPool"\);

if \(bundle == null\) {

throw new IllegalArgumentException\(

"\[jedisPool.properties\] is not found!"\);

}

JedisPoolConfig config = new JedisPoolConfig\(\);

config.setMaxTotal\(Integer.valueOf\(bundle

.getString\("redis.pool.maxTotal"\)\)\);

config.setMaxIdle\(Integer.valueOf\(bundle

.getString\("redis.pool.maxIdle"\)\)\);

config.setMaxWaitMillis\(Long.valueOf\(bundle.getString\("redis.pool.maxWait"\)\)\);

config.setTestOnBorrow\(Boolean.valueOf\(bundle

.getString\("redis.pool.testOnBorrow"\)\)\);

config.setTestOnReturn\(Boolean.valueOf\(bundle

.getString\("redis.pool.testOnReturn"\)\)\);

jedisPool = new JedisPool\(config, bundle.getString\("redis.ip"\),

Integer.valueOf\(bundle.getString\("redis.port"\)\)\);

// 从池中获取一个Jedis对象

jedis = jedisPool .getResource\(\);

}

public void add\(String sn\) {

jedis.sadd\("snSet", sn\);

jedisPool.destroy\(\);

}

public void remove\(String sn\) {

jedis.srem\("snSet", sn\);

}

public boolean isExist\(String sn\) {

Set

&lt;

String

&gt;

 snSet = jedis.smembers\("snSet"\);

return snSet.contains\(sn\);

}

public static void main\(String\[\] args\) {

String keys = "name";

// 删数据

jedis.del\(keys\);

// 存数据

jedis.set\(keys, "snowolf"\);

// 取数据

String value = jedis.get\(keys\);

System.out.println\(value\);

// 释放对象池

// jedisPool.returnResource\(jedis\);

}

}

  


而直接通过代码实现的话，其实也是一个原理：

public class JedisUtil1 {

private static JedisUtil1 instance = null;

private Jedis jedis;

private static JedisPool jedisPool = null;

//Redis服务器IP

private static String HOST = "172.24.4.183";

//Redis的端口号

private static int PORT = 6379;

//可用连接实例的最大数目，默认值为8；

//如果赋值为-1，则表示不限制；如果pool已经分配了maxActive个jedis实例，则此时pool的状态为exhausted\(耗尽\)。

private static int MAX\_ACTIVE = 1024;

//控制一个pool最多有多少个状态为idle\(空闲的\)的jedis实例，默认值也是8。

private static int MAX\_IDLE = 200;

private static int TIMEOUT = 10000;

//在borrow一个jedis实例时，是否提前进行validate操作；如果为true，则得到的jedis实例均是可用的；

private static boolean TEST\_ON\_BORROW = true;

/\*\*

\* 初始化Redis连接池

\*/

static {

try {

JedisPoolConfig config = new JedisPoolConfig\(\);

config.setMaxTotal\(MAX\_ACTIVE\);

config.setMaxIdle\(MAX\_IDLE\);

config.setMaxWaitMillis\(TIMEOUT\);

config.setTestOnBorrow\(TEST\_ON\_BORROW\);

jedisPool = new JedisPool\(config, HOST, PORT\);

} catch \(Exception e\) {

e.printStackTrace\(\);

}

}

* Sentinel连接池连接

Sentinel连接池用于应对Redis的Sentinel的主从切换机制，能够正确在服务器宕机导致服务器切换时得到正确的服务器连接，当服务器采用该部署策略的时候推荐使用该连接池进行操作

private static Jedis jedis;

private static JedisSentinelPool jedisSentinelPool = null;

  


/\*\*

\* 初始化Redis连接池

\*/

static {

ResourceBundle bundle = ResourceBundle.getBundle\("jedisSentinePool"\);

if \(bundle == null\) {

throw new IllegalArgumentException\(

"\[jedisSentinePool.properties\] is not found!"\);

}

JedisPoolConfig config = new JedisPoolConfig\(\);

config.setMaxTotal\(Integer.valueOf\(bundle

.getString\("redis.pool.maxTotal"\)\)\);

config.setMaxIdle\(Integer.valueOf\(bundle

.getString\("redis.pool.maxIdle"\)\)\);

config.setMaxWaitMillis\(Long.valueOf\(bundle.getString\("redis.pool.maxWait"\)\)\);

config.setTestOnBorrow\(Boolean.valueOf\(bundle

.getString\("redis.pool.testOnBorrow"\)\)\);

config.setTestOnReturn\(Boolean.valueOf\(bundle

.getString\("redis.pool.testOnReturn"\)\)\);

//监听器列表

Set

&lt;

String

&gt;

 sentinels = new HashSet

&lt;

&gt;

\(\);

/\*\*

\*

//监听器1

sentinels.add\(new HostAndPort\("172.24.4.183", 26379\).toString\(\)\);

//监听器2

sentinels.add\(new HostAndPort\("172.24.4.184", 26379\).toString\(\)\);

//实际使用的时候在properties里配置即可：redis.sentinel.hostandports=172.24.4.183:26379, 172.24.4.184:26379

//然后使用 bundle.getString\("redis.sentinel.hostandports"\);获取地址

\*/

//mastername是服务器上的master的名字，在master服务器的sentinel.conf中配置

String masterName = bundle.getString\("redis.sentinel.masterName"\);

sentinels.add\(bundle.getString\("redis.sentinel.hostandports"\)\);

//初始化连接池

jedisSentinelPool = new JedisSentinelPool\(masterName,

sentinels, config\);

// 从池中获取一个Jedis对象

jedis = jedisSentinelPool.getResource\(\);

}

* ShardedJedisPool连接池分片连接

Memcached完全基于分布式集群，而Redis是Master-Slave，Redis在容灾处理方面可以通过服务器端配置Master-Slave模式来实现。如果想把Reids做成集群模式，无外乎多做几套Master-Slave，每套Master-Slave完成各自的容灾处理，通过Client工具来实现一致性哈希分布存储，即key分片存储。

shared一致性哈希采用以下方案：

1. Redis服务器节点划分：将每台服务器节点采用hash算法划分为160个虚拟节点\(可以配置划分权重\)
2. 将划分虚拟节点采用TreeMap存储
3. 对每个Redis服务器的物理连接采用LinkedHashMap存储
4. 对Key or KeyTag 采用同样的hash算法，然后从TreeMap获取大于等于键hash值得节点，取最邻近节点存储；当key的hash值大于虚拟节点hash值得最大值时，存入第一个虚拟节点

sharded采用的hash算法：MD5 和 MurmurHash两种；默认采用64位的MurmurHash算法；有兴趣的可以研究下~

保留前面的JedisPoolConfig，新增两个Redis的IP（redis1.ip，redis2.ip），完成两个JedisShardInfo实例，并将其丢进List中：

private static ShardedJedis jedis;

private static ShardedJedisPool shardedJedisPool = null;

/\*\*

\* 初始化Redis连接池

\*/

static {

ResourceBundle bundle = ResourceBundle.getBundle\("sharedJedisPool"\);

if \(bundle == null\) {

throw new IllegalArgumentException\(

"\[jedisPool.properties\] is not found!"\);

}

JedisPoolConfig config = new JedisPoolConfig\(\);

config.setMaxTotal\(Integer.valueOf\(bundle

.getString\("redis.pool.maxTotal"\)\)\);

config.setMaxIdle\(Integer.valueOf\(bundle

.getString\("redis.pool.maxIdle"\)\)\);

config.setMaxWaitMillis\(Long.valueOf\(bundle.getString\("redis.pool.maxWait"\)\)\);

config.setTestOnBorrow\(Boolean.valueOf\(bundle

.getString\("redis.pool.testOnBorrow"\)\)\);

config.setTestOnReturn\(Boolean.valueOf\(bundle

.getString\("redis.pool.testOnReturn"\)\)\);

JedisShardInfo jedisShardInfo1 = new JedisShardInfo\(

bundle.getString\("redis1.ip"\), Integer.valueOf\(bundle.getString\("redis.port"\)\)\);

JedisShardInfo jedisShardInfo2 = new JedisShardInfo\(

bundle.getString\("redis2.ip"\), Integer.valueOf\(bundle.getString\("redis.port"\)\)\);

List

&lt;

JedisShardInfo

&gt;

 list = new LinkedList

&lt;

&gt;

\(\);

list.add\(jedisShardInfo1\);

list.add\(jedisShardInfo2\);

shardedJedisPool = new ShardedJedisPool\(config,list\);

// 从池中获取一个Jedis对象

jedis = shardedJedisPool.getResource\(\);

}

参考文档：

[https://blog.csdn.net/xuemengrui12/article/details/75097715](https://blog.csdn.net/xuemengrui12/article/details/75097715)







