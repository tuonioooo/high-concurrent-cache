# Redis的bitmap讲解

### 基本语法：

**1）SETBIT**

```
redis 127.0.0.1:6379> setbit KEY_NAME OFFSET VALUE 
//该命令用于对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。时间复杂度O（1）
```

在redis中，存储的字符串都是以二进制的形式存在的。比如：设置一个key-value，键的名字叫“andy” ，值为字符’a’，‘a’ 的ASCII码是97。转换为二进制是：01100001。offset的学名叫做“偏移” ，二进制中的每一位就是offset值，比如在这里offset 0 等于 ‘0’ ，offset 1等于’1’ ，offset2等于’1’，offset 6 等于’1’ ，没错，offset是从左往右计数的，也就是从高位往低位。

那如何通过SETBIT命令将 andy中的 ‘a’ 变成 ‘b’ 呢？即将 01100001 变成 01100010（b的ASCII码是98），其实就是将’a’中的offset 6从0变成1，将offset 7从1变成0。

![](/assets/20180106165624903.png)

每次SETBIT完毕之后，有一个（integer） 0或者（integer）1的返回值，这个是在你进行SETBIT 之前，该offset位的比特值。最后通过get andy得到的结果变成了 ‘b’ 。

**2）BITCOUNT**

```
redis 127.0.0.1:6379> bitcount andy 
//该命令统计字符串（字节）被设置为1的bit数
```

经过setbit操作之后，andy代表的01100010（b的ASCII码是98），共有3个1。

![](/assets/20180106165714082.png)

> bitcount 统计的是1的个数， bitcount test 0 -1 就是所有的， bitcount 0 0 那么就应该是第一个字节中1的数量的，注意是字节 第一个字节也就是 0 1 2 3 4 5 6 7 这八个位置上。见下面的测试样例，setbit单位是bit，bitcount是以byte为间隔统计的

**3）GETBIT**

```
redis 127.0.0.1:6379> getbit andy offset  	//返回key对应的string在offset处的bit值
```

> 一个bitmap默认包含40亿个位

**4）BITOP**

```
redis 127.0.0.1:6379> bitop operation destkey key [key...]  
//对一个或多个保存二进制位的字符串 key 进行位元操作，并将结果保存到 destkey 上	
```

BITOP 命令支持 AND 、 OR 、 NOT 、 XOR 这四种操作中的任意一种参数：



BITOP AND destkey srckey1 … srckeyN ，对一个或多个 key 求逻辑与，并将结果保存到 destkey

BITOP OR destkey srckey1 … srckeyN，对一个或多个 key 求逻辑或，并将结果保存到 destkey

BITOP XOR destkey srckey1 … srckeyN，对一个或多个 key 求逻辑异或，并将结果保存到 destkey

BITOP NOT destkey srckey，对给定 key 求逻辑非，并将结果保存到 destkey

  除了 NOT 操作之外，其他操作都可以接受一个或多个 key 作为输入，执行结果将始终保持到destkey里面。



  当 BITOP 处理不同长度的字符串时，较短的那个字符串所缺少的部分会被看作 0。返回值是保存到 destkey 的字符串的长度（以字节byte为单位），和输入 key 中最长的字符串长度相等。



