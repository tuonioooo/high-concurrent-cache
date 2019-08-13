# Redis的bitmap讲解

### 基本语法：

1）SETBIT

```
redis 127.0.0.1:6379> setbit KEY_NAME OFFSET VALUE 
//该命令用于对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。时间复杂度O（1）
```

在redis中，存储的字符串都是以二进制的形式存在的。比如：设置一个key-value，键的名字叫“andy” ，值为字符’a’，‘a’ 的ASCII码是97。转换为二进制是：01100001。offset的学名叫做“偏移” ，二进制中的每一位就是offset值，比如在这里offset 0 等于 ‘0’ ，offset 1等于’1’ ，offset2等于’1’，offset 6 等于’1’ ，没错，offset是从左往右计数的，也就是从高位往低位。

那如何通过SETBIT命令将 andy中的 ‘a’ 变成 ‘b’ 呢？即将 01100001 变成 01100010（b的ASCII码是98），其实就是将’a’中的offset 6从0变成1，将offset 7从1变成0。

