# 排行榜

* ## ZADD

**ZADD key score member \[\[score member\] \[score member\] ...\]**

将一个或多个member元素及其score值加入到有序集key当中。

如果某个member已经是有序集的成员，那么更新这个member的score值，并通过重新插入这个member元素，来保证该member在正确的位置上。

score值可以是整数值或双精度浮点数。

如果key不存在，则创建一个空的有序集并执行[ZADD](http://doc.redisfans.com/sorted_set/zadd.html#zadd)操作。

当key存在但不是有序集类型时，返回一个错误。

对有序集的更多介绍请参见[sorted set](http://redis.io/topics/data-types#sorted-sets)。

在 Redis 2.4 版本以前，[ZADD](http://doc.redisfans.com/sorted_set/zadd.html#zadd)每次只能添加一个元素。

**可用版本：**

&gt;= 1.2.0

**时间复杂度:**

O\(M\*log\(N\)\)， N 是有序集的基数， M 为成功添加的新成员的数量。

**返回值:**

被成功添加的新成员的数量，不包括那些被更新的、已经存在的成员。

* ## ZRANGE

**ZRANGE key start stop \[WITHSCORES\]**

返回有序集key中，指定区间内的成员。

其中成员的位置按score值递增\(从小到大\)来排序。

具有相同score值的成员按字典序\([lexicographical order](http://en.wikipedia.org/wiki/Lexicographical_order)\)来排列。

如果你需要成员按score值递减\(从大到小\)来排列，请使用[_ZREVRANGE_](http://doc.redisfans.com/sorted_set/zrevrange.html#zrevrange)命令。

下标参数 start 和 stop 都以 0 为底，也就是说，以 0 表示有序集第一个成员，以 1 表示有序集第二个成员，以此类推。

你也可以使用负数下标，以 -1 表示最后一个成员， -2 表示倒数第二个成员，以此类推。

超出范围的下标并不会引起错误。

比如说，当 start 的值比有序集的最大下标还要大，或是 start &gt; stop 时， ZRANGE 命令只是简单地返回一个空列表。

另一方面，假如 stop 参数的值比有序集的最大下标还要大，那么 Redis 将 stop 当作最大下标来处理。

可以通过使用 WITHSCORES 选项，来让成员和它的 score 值一并返回，返回列表以 value1,score1, ..., valueN,scoreN 的格式表示。

客户端库可能会返回一些更复杂的数据类型，比如数组、元组等。

可用版本：

&gt;= 1.2.0

时间复杂度:

O\(log\(N\)+M\)， N 为有序集的基数，而 M 为结果集的基数。

返回值:

指定区间内，带有 score 值\(可选\)的有序集成员的列表。

* ## ZRANK

**ZRANK key member**

返回有序集key中成员member的排名。其中有序集成员按score值递增\(从小到大\)顺序排列。

排名以0为底，也就是说，score值最小的成员排名为0。

使用[_ZREVRANK_](http://doc.redisfans.com/sorted_set/zrevrank.html#zrevrank)命令可以获得成员按score值递减\(从大到小\)排列的排名。

可用版本：

&gt;= 2.0.0

时间复杂度:

O\(log\(N\)\)

返回值:

如果 member 是有序集 key 的成员，返回 member 的排名。

如果 member 不是有序集 key 的成员，返回 nil 。

* ## 相关排行榜示例

```
# 添加单个元素

redis> ZADD page_rank 10 google.com
(integer) 1


# 添加多个元素

redis> ZADD page_rank 9 baidu.com 8 bing.com
(integer) 2

redis> ZRANGE page_rank 0 -1 WITHSCORES
1) "bing.com"
2) "8"
3) "baidu.com"
4) "9"
5) "google.com"
6) "10"


# 添加已存在元素，且 score 值不变

redis> ZADD page_rank 10 google.com
(integer) 0

redis> ZRANGE page_rank 0 -1 WITHSCORES  # 没有改变
1) "bing.com"
2) "8"
3) "baidu.com"
4) "9"
5) "google.com"
6) "10"


# 添加已存在元素，但是改变 score 值

redis> ZADD page_rank 6 bing.com
(integer) 0

redis> ZRANGE page_rank 0 -1 WITHSCORES  # bing.com 元素的 score 值被改变
1) "bing.com"
2) "6"
3) "baidu.com"
4) "9"
5) "google.com"
6) "10"
```

```
redis > ZADD salary 10086 boss 5000 tom 3500 jack # 创建有序集合序列
(integer) 3

redis > ZRANGE salary 0 -1 WITHSCORES             # 显示整个有序集成员
1) "jack"
2) "3500"
3) "tom"
4) "5000"
5) "boss"
6) "10086"

redis > ZRANGE salary 1 2 WITHSCORES              # 显示有序集下标区间 1 至 2 的成员
1) "tom"
2) "5000"
3) "boss"
4) "10086"

redis > ZRANGE salary 0 200000 WITHSCORES         # 测试 end 下标超出最大下标时的情况
1) "jack"
2) "3500"
3) "tom"
4) "5000"
5) "boss"
6) "10086"

redis > ZRANGE salary 200000 3000000 WITHSCORES   # 测试当给定区间不存在于有序集时的情况
(empty list or set)
```

```
127.0.0.1:6379> ZRANGE salary 0 -1 WITHSCORES  # 显示所有成员及其 score 值
1) "peter"
2) "3500"
3) "tom"
4) "4000"
5) "jack"
6) "5000"

127.0.0.1:6379> ZRANK salary tom    # 显示 tom 的薪水排名，第二
(integer) 1
```

* ## **Java 应用示例**

User.java

```
package com.duobei.model;

import java.io.Serializable;

public class User implements Serializable {
    private static final long serialVersionUID = 1L;

    private String id;                //编号
    private String name;            //姓名
    private double score;            //得分
    private int rank;                //排名

    public User() {

    }

    public User(String id, String name, double score) {
        this.id = id;
        this.name = name;
        this.score = score;
    }

    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public double getScore() {
        return score;
    }
    public void setScore(double score) {
        this.score = score;
    }
    public int getRank() {
        return rank;
    }
    public void setRank(int rank) {
        this.rank = rank;
    }

    @Override
    public String toString() {
        return "User [id=" + id + ", name=" + name + ", score=" + score
                + ", rank=" + rank + "]";
    }

}
```

自定义序列化封装类

```
package com.duobei.tools;
 
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
 
public class ObjectSer {
 
	/**
	 * 对象序列化
	 * @param obj
	 * @return
	 */
	public static byte[] ObjectToByte(Object obj) {
		byte[] bytes = null;
		try {
			ByteArrayOutputStream bo = new ByteArrayOutputStream();
			ObjectOutputStream oo = new ObjectOutputStream(bo);
			oo.writeObject(obj);
			bytes = bo.toByteArray();
			bo.close();
			oo.close();  
		}
		catch(Exception e) { 
			e.printStackTrace();
		}
		return bytes;
    }
	
	/**
	 * 反序列化
	 * @param bytes
	 * @return
	 */
	public static Object ByteToObject(byte[] bytes) {
		Object object = null;
		try {
			ByteArrayInputStream bais = new ByteArrayInputStream(bytes);
			ObjectInputStream ois = new ObjectInputStream(bais);
			object = ois.readObject();
		} catch (Exception e) {
			e.printStackTrace();
		}
		return object;
	}
}

```

测试类

```
package com.duobei.test;
 
 
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Set;
 
import org.junit.Before;
import org.junit.Ignore;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
import redis.clients.jedis.ShardedJedis;
import redis.clients.jedis.ShardedJedisPool;
 
import com.duobei.model.User;
import com.duobei.tools.ObjectSer;
 
public class RankingTest {
 
	private ApplicationContext context;
	private ShardedJedisPool shardedJedisPool;
	private ShardedJedis jedis;
 
	public RankingTest() {
 
	}
 
	@Before
	public void init() throws Exception {
 
		String config[] = { "applicationContext.xml" };
		context = new ClassPathXmlApplicationContext(config);
 
		shardedJedisPool = (ShardedJedisPool) context.getBean("shardedJedisPool");
		jedis = (ShardedJedis) shardedJedisPool.getResource();
		
	}
	
	@Test
	@Ignore
	public void rankAdd() {
		User user1 = new User("12345", "常少鹏", 99.9);
		User user2 = new User("12346", "王卓卓", 99.8);
		User user3 = new User("12347", "邹雨欣", 96.8);
		User user4 = new User("12348", "郑伟山", 98.8);
		User user5 = new User("12349", "李超杰", 99.6);
		User user6 = new User("12350", "董明明", 99.0);
		User user7 = new User("12351", "陈国峰", 100.0);
		User user8 = new User("12352", "楚晓丽", 99.6);
		jedis.zadd("game".getBytes(), user1.getScore(), ObjectSer.ObjectToByte(user1));
		jedis.zadd("game".getBytes(), user2.getScore(), ObjectSer.ObjectToByte(user2));
		jedis.zadd("game".getBytes(), user3.getScore(), ObjectSer.ObjectToByte(user3));
		jedis.zadd("game".getBytes(), user4.getScore(), ObjectSer.ObjectToByte(user4));
		jedis.zadd("game".getBytes(), user5.getScore(), ObjectSer.ObjectToByte(user5));
		jedis.zadd("game".getBytes(), user6.getScore(), ObjectSer.ObjectToByte(user6));
		jedis.zadd("game".getBytes(), user7.getScore(), ObjectSer.ObjectToByte(user7));
		jedis.zadd("game".getBytes(), user8.getScore(), ObjectSer.ObjectToByte(user8));
	}
		
	@Test
	//@Ignore
	public void gameRankShow() {
		Set<byte[]> set = jedis.zrevrange("game".getBytes(), 0, -1);
		Iterator<byte[]> iter = set.iterator();
	
		int i = 1;
		List<User> list = new ArrayList<User>();
		while(iter.hasNext()) {
			User user = (User) ObjectSer.ByteToObject(iter.next());
			user.setRank(i++);
			list.add(user);
		}
		
		for(User user : list) 
			System.out.println(user);
	}
	
}

```

测试结果

```
User [id=12351, name=陈国峰, score=100.0, rank=1]
User [id=12345, name=常少鹏, score=99.9, rank=2]
User [id=12346, name=王卓卓, score=99.8, rank=3]
User [id=12352, name=楚晓丽, score=99.6, rank=4]
User [id=12349, name=李超杰, score=99.6, rank=5]
User [id=12350, name=董明明, score=99.0, rank=6]
User [id=12348, name=郑伟山, score=98.8, rank=7]
User [id=12347, name=邹雨欣, score=96.8, rank=8]

```

* ## 补充

先关SortedSet集合请参考，官方文档：[http://doc.redisfans.com/sorted\_set/index.html](http://doc.redisfans.com/sorted_set/index.html)

