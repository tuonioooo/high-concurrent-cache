# Java客户端

使用 Java 程序连接 Memcached，需要在你的 classpath 中添加 Memcached jar 包。

本站 jar 包下载地址：[spymemcached-2.10.3.jar](http://www.runoob.com/try/download/spymemcached-2.10.3.jar)。

Google Code jar 包下载地址：[spymemcached-2.10.3.jar](http://code.google.com/p/spymemcached/downloads/list)（需要翻墙）。



以下程序假定 Memcached 服务的主机为 127.0.0.1，端口为 11211。

### 连接实例

Java 连接 Memcached

## MemcachedJava.java 文件：

```
import net.spy.memcached.MemcachedClient;
import java.net.*;
 
 
public class MemcachedJava {
   public static void main(String[] args) {
      try{
         // 本地连接 Memcached 服务
         MemcachedClient mcc = new MemcachedClient(new InetSocketAddress("127.0.0.1", 11211));
         System.out.println("Connection to server sucessful.");
         
         // 关闭连接
         mcc.shutdown();
         
      }catch(Exception ex){
         System.out.println( ex.getMessage() );
      }
   }
}
```

该程序中我们使用 InetSocketAddress 连接 IP 为 127.0.0.1 端口 为 11211 的 memcached 服务。

执行以上代码，如果连接成功会输出以下信息：

```
Connection to server successful.
```

### set 操作实例

以下使用 java.util.concurrent.Future 来存储数据

```
import java.net.InetSocketAddress;
import java.util.concurrent.Future;
 
import net.spy.memcached.MemcachedClient;
 
public class MemcachedJava {
   public static void main(String[] args) {
   
      try{
         // 连接本地的 Memcached 服务
         MemcachedClient mcc = new MemcachedClient(new InetSocketAddress("127.0.0.1", 11211));
         System.out.println("Connection to server sucessful.");
      
         // 存储数据
         Future fo = mcc.set("runoob", 900, "Free Education");
      
         // 查看存储状态
         System.out.println("set status:" + fo.get());
         
         // 输出值
         System.out.println("runoob value in cache - " + mcc.get("runoob"));
 
         // 关闭连接
         mcc.shutdown();
         
      }catch(Exception ex){
         System.out.println( ex.getMessage() );
      }
   }
}
```

执行程序，输出结果为：

```
Connection to server successful.
set status:true
runoob value in cache - Free Education
```

### add 操作实例

```
import java.net.InetSocketAddress;
import java.util.concurrent.Future;
 
import net.spy.memcached.MemcachedClient;
 
public class MemcachedJava {
   public static void main(String[] args) {
   
      try{
   
         // 连接本地的 Memcached 服务
         MemcachedClient mcc = new MemcachedClient(new InetSocketAddress("127.0.0.1", 11211));
         System.out.println("Connection to server sucessful.");
 
         // 添加数据
         Future fo = mcc.set("runoob", 900, "Free Education");
 
         // 打印状态
         System.out.println("set status:" + fo.get());
 
         // 输出
         System.out.println("runoob value in cache - " + mcc.get("runoob"));
 
         // 添加
         fo = mcc.add("runoob", 900, "memcached");
 
         // 打印状态
         System.out.println("add status:" + fo.get());
 
         // 添加新key
         fo = mcc.add("codingground", 900, "All Free Compilers");
 
         // 打印状态
         System.out.println("add status:" + fo.get());
         
         // 输出
         System.out.println("codingground value in cache - " + mcc.get("codingground"));
 
         // 关闭连接
         mcc.shutdown();
         
      }catch(Exception ex){
         System.out.println(ex.getMessage());
      }
   }
}
```

### replace 操作实例

```
import java.net.InetSocketAddress;
import java.util.concurrent.Future;
 
import net.spy.memcached.MemcachedClient;
 
public class MemcachedJava {
   public static void main(String[] args) {
   
      try {
         //连接本地的 Memcached 服务
         MemcachedClient mcc = new MemcachedClient(new InetSocketAddress("127.0.0.1", 11211));
         System.out.println("Connection to server sucessful.");
 
         // 添加第一个 key=》value 对
         Future fo = mcc.set("runoob", 900, "Free Education");
 
         // 输出执行 add 方法后的状态
         System.out.println("add status:" + fo.get());
 
         // 获取键对应的值
         System.out.println("runoob value in cache - " + mcc.get("runoob"));
 
         // 添加新的 key
         fo = mcc.replace("runoob", 900, "Largest Tutorials' Library");
 
         // 输出执行 set 方法后的状态
         System.out.println("replace status:" + fo.get());
 
         // 获取键对应的值
         System.out.println("runoob value in cache - " + mcc.get("runoob"));
 
         // 关闭连接
         mcc.shutdown();
         
      }catch(Exception ex){
         System.out.println( ex.getMessage() );
      }
   }
}
```

### append 操作实例

```
import java.net.InetSocketAddress;
import java.util.concurrent.Future;
 
import net.spy.memcached.MemcachedClient;
 
public class MemcachedJava {
   public static void main(String[] args) {
   
      try{
   
         // 连接本地的 Memcached 服务
         MemcachedClient mcc = new MemcachedClient(new InetSocketAddress("127.0.0.1", 11211));
         System.out.println("Connection to server sucessful.");
 
         // 添加数据
         Future fo = mcc.set("runoob", 900, "Free Education");
 
         // 输出执行 set 方法后的状态
         System.out.println("set status:" + fo.get());
 
         // 获取键对应的值
         System.out.println("runoob value in cache - " + mcc.get("runoob"));
 
         // 对存在的key进行数据添加操作
         fo = mcc.append("runoob", 900, " for All");
 
         // 输出执行 set 方法后的状态
         System.out.println("append status:" + fo.get());
         
         // 获取键对应的值
         System.out.println("runoob value in cache - " + mcc.get("codingground"));
 
         // 关闭连接
         mcc.shutdown();
         
      }catch(Exception ex) {
         System.out.println(ex.getMessage());
      ]
   }
}
```



