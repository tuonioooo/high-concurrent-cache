# Memcache 高可用集群之magent实现主从

Magent 是一款开源的 Memcached 代理服务器软件，使用它可以搭建高可用性的集群应用的 Memcached 服务 ，备份 Memcached 数据，尽管 Memcached 服务挂掉，前端也能获取到数据，客户端先连到 Magent 代理服务器 ，然后Magent 代理服务器 在可以连接多台 Memcached 服务器，然后可以进行数据的保存和备份数据。这样数据就不会丢失，保存了数据完整性。

项目地址：

[http://code.google.com/p/memagent](http://code.google.com/p/memagent)

[https://github.com/wangmh/memagent](https://github.com/wangmh/memagent)

  


**一、整体架构**

![](/assets/import-memcached-003.png)  


从图中可以看到有两个magent节点，两个memcached节点，每个magent节点又分别代理两个memcached节点，应用系统端使用magent pool来调用memcache进行存储。硬件结构为两台linux服务器，每台服务器上分别安装magent和memcached服务，并设为开机启动。这样做的好处是任何一台服务器宕机后都不影响magent pool获取memcache信息，即实现了memcached的高可用（HA），如果两台机器都宕机了，只能说明你RP太差了。当然，也可以用三台、四台或者更多服务器来提高HA。

**二、安装**

**memcached和Magent**

**1.安装memcached**，请参考：[http://blog.csdn.net/zhu\_tianwei/article/details/44542497](http://blog.csdn.net/zhu_tianwei/article/details/44542497)

**2.安装Magent**

1）下载安装

wget [http://memagent.googlecode.com/files/magent-0.6.tar.gz](http://memagent.googlecode.com/files/magent-0.6.tar.gz)

tar -zxvf magent-0.6.tar.gz

make

检查是否安装成功：

./magent -h 

memcached agent v0.6 Build-Date: Mar 27 2015 07:09:30

错误修改：

**1）错误1**

gcc -Wall -g -O2 -I/usr/local/include  -c -o magent.o magent.c

magent.c: In function ‘writev\_list’:

magent.c:729: error: ‘SSIZE\_MAX’ undeclared \(first use in this function\)

magent.c:729: error: \(Each undeclared identifier is reported only once

magent.c:729: error: for each function it appears in.\)

make: \*\*\* \[magent.o\] Error 1

处理，在ketama.h开头添加

\#ifndef SSIZE\_MAX

\#define SSIZE\_MAX      32767

\#endif

**2）错误2**

gcc -Wall -g -O2 -I/usr/local/include -m64 -c -o magent.o magent.c

gcc -Wall -g -O2 -I/usr/local/include -m64 -c -o ketama.o ketama.c

gcc -Wall -g -O2 -I/usr/local/include -m64 -o magent magent.o ketama.o /usr/lib64/libevent.a /usr/lib64/libm.a

/usr/lib64/libevent.a\(event.o\): In function \`gettime’:

\(.text+0×449\): undefined reference to \`clock\_gettime’

/usr/lib64/libevent.a\(event.o\): In function \`event\_base\_new’:

\(.text+0x72a\): undefined reference to \`clock\_gettime’

collect2: ld returned 1 exit status

make: \*\*\* \[magent\] Error 1

处理：

vim Makefile

CFLAGS = -Wall -g -O2 -I/usr/local/include $\(M64\)

改为：    

CFLAGS = -lrt -Wall -g -O2 -I/usr/local/include $\(M64\)

**3）错误3**

gcc -Wall -g -O2 -I/usr/local/include -m64 -c -o magent.o magent.c

gcc -Wall -g -O2 -I/usr/local/include -m64 -c -o ketama.o ketama.c

gcc -Wall -g -O2 -I/usr/local/include -m64 -o magent magent.o ketama.o /usr/lib64/libevent.a /usr/lib64/libm.a

gcc: /usr/lib64/libm.a：没有那个文件或目录

make: \*\*\* \[magent\] 错误 1

解决办法

ln -s /usr/lib64/libm.so /usr/lib64/libm.a

注：有可能还会报错 gcc: /usr/lib64/libevent.a: 没有那个文件或目录

如果有，可执行

vi Makefile

找到 LIBS = /usr/lib64/libevent.a /usr/lib64/libm.a

修改 LIBS = /usr/libevent 的安装路径/libevent.a /usr/lib64/libm.a

例： LIBS = /usr/lib/libevent.a /usr/lib64/libm.a

**三、启动服务**

**1.memcached**

192.168.36.54:11211

192.168.36.189:11211

./memcached/bin/memcached -d -m 64 -p 11211

**2.magent**

192.168.36.54

 ./magent/magent -n 4096  -l 192.168.36.54 -p 11200 -s 192.168.36.54:11211 -b 192.168.36.189:11211

192.168.36.189

./magent/magent -n 4096  -l 192.168.36.189 -p 11200 -s 192.168.36.189:11211 -b 192.168.36.54:11211

magent参数说明：

  -h 帮助说明

  -u 用户

  -g gid

  -p 启动端口, 默认11211. \(0 to disable tcp support\)

  -s 服务memcached地址，ip:port, set memcached server ip and port

  -b 备份memcached地址，ip:port, set backup memcached server ip and port

  -l 启动IP地址，ip, local bind ip address, default is 0.0.0.0

  -n 最大并发数number, set max connections, default is 4096

  -D 非后台运行don't go to background

  -k use ketama key allocation algorithm

  -f file, unix socket path to listen on. default is off

  -i number, set max keep alive connections for one memcached server, default is 20

  -v verbose

**四、测试**

1.通过代理magent添加数据，查看服务和备份memcached

通过任何一个代理magent添加数据，2台memcached都存在。

2.关闭一个服务memcached，再重启查看数据是否存在

关闭192.168.36.189的memcached，通过192.168.36.189的magent可以查询数据，来自备份机器。

再启动，由于没有持久化，所有数据都无法查询，另外一台magent正常。

再启动192.168.36.189的memcached的时候，

可以从备份Memcached服务恢复数据，起到了容错的作用。

**通过以上测试可以得出结论**:

1、通过magent的连接池放的值会分别存在magent代理的所有memcached上去。

2、如果有一个memcached宕机通过magent代理方式还能取到值。

3、如果memcached修复重启后通过magent代理方式取到的值就会为Null，这是由于memcache重启后里边的值随着memcache服务的停止就消失了（因为在内存中），但是magent是通过key进行哈希计算分配到某台机器上的，memcache重启后会还从这台机器上取值，所有取到的值就没空。

解决办法

1、在每次memcache宕机修复后可以写一个程序把集群中的其他memcache的所有信息全给拷贝到当前宕机修复后的memcache中。

2、自己写代理，当从一个memcached服务上取到的值为null时再去其他memcached上取值。

注意事项

magent的调用方式同memcached一样，客户端可以不用改代码即可实现切换到magent模式下。

  


