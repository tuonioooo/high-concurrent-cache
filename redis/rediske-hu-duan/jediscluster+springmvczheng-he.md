# jedisCluster+SpringMVC整合

## maven依赖 {#maven依赖}

springboot整合jedisCluster相当简单，maven依赖如下:

```
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-redis</artifactId>
 </dependency>
```

加了这一个依赖之后就不要再加上jedis的这一个依赖了：

```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

加这个可能在本身测试的时候，可能会导致jedisCluster对象正常，但是在测试的时候会发现set数据的时候会出现问题，我把jedis的依赖去掉之后，这个问题解决，因此不要加上jedis的这一个依赖，spring-boot-starter-redis这一个引入相关jedis需要的包。

## application.properties配置 {#applicationproperties配置}

这里的配置相当简单，只需要天上redis的相关地址就行了，如下：

```
#redis cluster
spring.redis.cache.clusterNodes=192.168.xx.xx:6379,192.168.xx.:6380,192.168.xx.xx:6381
spring.redis.cache.commandTimeout=5000
```

定义一个类命名问RedisProperties，在里面定义的字段与配置文件中相对应，即可取到配置，如下：

```
@Component
@ConfigurationProperties(prefix = "spring.redis.cache")
@Data
public class RedisProperties {

    private String clusterNodes;
    private Integer   commandTimeout;
}
```

## JedisClusterConfig {#jedisclusterconfig}

```
@Configuration
@ConditionalOnClass({JedisCluster.class})
@EnableConfigurationProperties(RedisProperties.class)
public class JedisClusterConfig {

    @Inject
    private RedisProperties redisProperties;

    @Bean
    @Singleton
    public JedisCluster getJedisCluster() {
        String[] serverArray = redisProperties.getClusterNodes().split(",");
        Set<HostAndPort> nodes = new HashSet<>();
        for (String ipPort: serverArray) {
            String[] ipPortPair = ipPort.split(":");
            nodes.add(new HostAndPort(ipPortPair[0].trim(),Integer.valueOf(ipPortPair[1].trim())));
        }
        return new JedisCluster(nodes, redisProperties.getCommandTimeout());
    }
}
```

配置就完成，现在进行测试一次。

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = SpringBootWebApplication.class)
@WebAppConfiguration
public class TestJedisCluster {

    @Inject
    private JedisCluster jedisCluster;

    @Test
    public void testJedis() {
        jedisCluster.set("test_jedis_cluster", "38967");
        Assert.assertEquals("38967", jedisCluster.get("test_jedis_cluster"));
        jedisCluster.del("test_jedis_cluster");
    }
}
```

使用RedisTemplate，添加如下依赖：

```
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-data-redis</artifactId>  
</dependency>  
```

配置文件application.yml在添加配置（假设有6个nodes）：

```
spring:  
  redis:  
    cluster:  
      nodes:  
        - 192.168.0.17:6390  
        - 192.168.0.17:6391  
        - 192.168.0.17:6392  
        - 192.168.0.9:6390  
        - 192.168.0.9:6391  
        - 192.168.0.9:6392
```

代码测试

```
@Autowired  
RedisTemplate<String, String> redisTemplate;  
  
@Test  
public void redisTest() {  
    String key = "redisTestKey";  
    String value = "I am test value";  
      
    ValueOperations<String, String> opsForValue = redisTemplate.opsForValue();  
      
    //数据插入测试：  
    opsForValue.set(key, value);  
    String valueFromRedis = opsForValue.get(key);  
    logger.info("redis value after set: {}", valueFromRedis);  
    assertThat(valueFromRedis, is(value));  
      
    //数据删除测试：  
    redisTemplate.delete(key);  
    valueFromRedis = opsForValue.get(key);  
    logger.info("redis value after delete: {}", valueFromRedis);  
    assertThat(valueFromRedis, equalTo(null));  
}
```



