# jedisCluster详解

Redis集群的搭建参考[Redis Cluster安装配置](http://06peng.com/archives/174)。

本文主要介绍JedisCluster客户端结合spring注解的使用。

> 1、rediscluster.properties

```
#redis的服务器地址
redis.host=172.18.209.168
#redis的服务端口
redis.port=6379
#密码
redis.password=×××××××
#最大空闲数
redis.maxIdle=100
#最大连接数
redis.maxActive=300
#最大建立连接等待时间
redis.maxWait=1000
#客户端超时时间单位是毫秒
redis.timeout=100000
redis.maxTotal=1000
redis.minIdle=8
#明是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个
redis.testOnBorrow=true
#连接耗尽时是否阻塞  false报异常,ture阻塞直到超时
redis.blockWhenExhausted=false
#当设置为true,且服务开启的jmx服务时,使用jconsole等工具将看到连接池的状态
redis.jmxEnabled=true
#向连接池归还链接时，是否检测链接对象的有效性
redis.testOnReturn=false
#对象空闲多久后逐出,当空闲时间>该值 且 空闲连接>最大空闲数 时直接逐出
redis.softMinEvictableIdleTimeMillis=10000

#jediscluster
cluster1.host.port=172.18.209.168:7001
cluster2.host.port=172.18.209.168:7002
cluster3.host.port=172.18.209.168:7003
cluster4.host.port=172.18.209.169:7004
cluster5.host.port=172.18.209.169:7005
cluster6.host.port=172.18.209.169:7006

#rediscluster
spring.redis.cluster.nodes=172.18.209.168:7001,172.18.209.168:7002,172.18.209.168:7003,172.18.209.169:7004,172.18.209.169:7005,172.18.209.169:7006
spring.redis.cluster.max-redirects=3  

```

> 2、spring-redis.xml

```
<bean id="propertyConfigurer"
      class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="location" value="classpath:rediscluster.properties" />
</bean>

<!-- 基础参数配置 -->
<bean id="jedisClusterPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <!-- 连接池最大允许连接数 -->
    <property name="maxTotal" value="${redis.maxTotal}"/>
    <!-- 对象空闲多久后逐出,当空闲时间>该值 且 空闲连接>最大空闲数 时直接逐出 -->
    <property name="softMinEvictableIdleTimeMillis" value="${redis.softMinEvictableIdleTimeMillis}"/>
    <!-- 向连接池归还链接时，是否检测链接对象的有效性 -->
    <property name="testOnReturn" value="${redis.testOnReturn}"/>
    <!-- 当设置为true,且服务开启的jmx服务时,使用jconsole等工具将看到连接池的状态 -->
    <property name="jmxEnabled" value="${redis.jmxEnabled}"/>
    <!-- 连接耗尽时是否阻塞  false报异常,ture阻塞直到超时 -->
    <property name="blockWhenExhausted" value="${redis.blockWhenExhausted}"/>
    <!--最大空闲数-->
    <property name="maxIdle" value="${redis.maxIdle}" />
    <!--最大建立连接等待时间-->
    <property name="maxWaitMillis" value="${redis.maxWait}" />
    <!--是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个-->
    <property name="testOnBorrow" value="${redis.testOnBorrow}" />
</bean>

<bean id="resourcePropertySource" class="org.springframework.core.io.support.ResourcePropertySource">
    <constructor-arg name="name" value="rediscluster.properties"/>
    <constructor-arg name="resource" value="classpath:rediscluster.properties"/>
</bean>

<bean id="redisClusterConfiguration" class="org.springframework.data.redis.connection.RedisClusterConfiguration">
    <constructor-arg name="propertySource" ref="resourcePropertySource"/>
</bean>

<!-- JedisCluster -->
<bean id="jedisCluster" class="*.*.jedis.JedisClusterFactory">
    <property name="addressConfig" value="classpath:rediscluster.properties" />
    <property name="addressKeyPrefix" value="cluster" />
    <property name="timeout" value="300000" />
    <property name="maxRedirections" value="6" />
    <property name="genericObjectPoolConfig" ref="jedisClusterPoolConfig" />
</bean>

<!-- 配置redis connection -->
<bean id="redisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
    <constructor-arg ref="redisClusterConfiguration"/>
    <constructor-arg ref="jedisClusterPoolConfig"/>
</bean>

<!-- 数据以字符串存储  -->
<bean id="stringRedisSerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer"/>

<!-- 数据以字节流存储 -->
<bean id="jdkRedisSerializer" class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer"/>

<!--  redis String类型 访问模版本 -->
<bean id="stringRedisTemplate" class="org.springframework.data.redis.core.StringRedisTemplate">
    <property name="connectionFactory" ref="redisConnectionFactory"/>
</bean>

<!-- redis 访问模版 -->
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
    <property name="connectionFactory" ref="redisConnectionFactory"/>
    <!-- 在hash数据结构中，hash-key的序列化策略 -->
    <property name="hashKeySerializer" ref="stringRedisSerializer"/>
    <!-- 对于普通K-V操作时，key采取的序列化策略 -->
    <property name="keySerializer" ref="stringRedisSerializer"/>
    <!-- value采取的序列化策略 -->
    <property name="valueSerializer" ref="jdkRedisSerializer"/>
</bean>
```

> 3、JedisClusterFactory.java

```
public class JedisClusterFactory implements FactoryBean, InitializingBean {

    private Resource addressConfig;
    private String addressKeyPrefix;

    private JedisCluster jedisCluster;
    private Integer timeout;
    private Integer maxRedirections;
    private GenericObjectPoolConfig genericObjectPoolConfig;

    private Pattern p = Pattern.compile("^.+[:]\\d{1,5}\\s*$");

    @Override
    public JedisCluster getObject() throws Exception {
        return jedisCluster;
    }

    @Override
    public Class<? extends JedisCluster> getObjectType() {
        return (this.jedisCluster != null ? this.jedisCluster.getClass() : JedisCluster.class);
    }

    @Override
    public boolean isSingleton() {
        return true;
    }

    private Set parseHostAndPort() throws Exception {
        try {
            Properties prop = new Properties();
            prop.load(this.addressConfig.getInputStream());
            Set haps = new HashSet<>();
            for (Object key : prop.keySet()) {
                if (!((String) key).startsWith(addressKeyPrefix)) {
                    continue;
                }
                String val = (String) prop.get(key);
                boolean isIpPort = p.matcher(val).matches();
                if (!isIpPort) {
                    throw new IllegalArgumentException("ip 或 port 不合法");
                }
                String[] ipAndPort = val.split(":");
                HostAndPort hap = new HostAndPort(ipAndPort[0], Integer.parseInt(ipAndPort[1]));
                haps.add(hap);
            }

            return haps;
        } catch (IllegalArgumentException ex) {
            throw ex;
        } catch (Exception ex) {
            throw new Exception("解析jedis配置文件失败", ex);
        }
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        Set haps = this.parseHostAndPort();
        jedisCluster = new JedisCluster(haps, timeout, maxRedirections, genericObjectPoolConfig);
    }

    public void setAddressConfig(Resource addressConfig) {
        this.addressConfig = addressConfig;
    }

    public void setTimeout(int timeout) {
        this.timeout = timeout;
    }

    public void setMaxRedirections(int maxRedirections) {
        this.maxRedirections = maxRedirections;
    }

    public void setAddressKeyPrefix(String addressKeyPrefix) {
        this.addressKeyPrefix = addressKeyPrefix;
    }

    public void setGenericObjectPoolConfig(GenericObjectPoolConfig genericObjectPoolConfig) {
        this.genericObjectPoolConfig = genericObjectPoolConfig;
    }
}
```



