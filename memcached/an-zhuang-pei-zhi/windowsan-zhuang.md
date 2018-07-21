# Windows安装

官网上并未提供 Memcached 的 Windows 平台安装包，我们可以使用以下链接来下载，你需要根据自己的系统平台及需要的版本号点击对应的链接下载即可：

* 32位系统 1.2.5版本：
  [http://static.runoob.com/download/memcached-1.2.5-win32-bin.zip](http://static.runoob.com/download/memcached-1.2.5-win32-bin.zip)
* 32位系统 1.2.6版本：
  [http://static.runoob.com/download/memcached-1.2.6-win32-bin.zip](http://static.runoob.com/download/memcached-1.2.6-win32-bin.zip)
* 32位系统 1.4.4版本：
  [http://static.runoob.com/download/memcached-win32-1.4.4-14.zip](http://static.runoob.com/download/memcached-win32-1.4.4-14.zip)
* 64位系统 1.4.4版本：
  [http://static.runoob.com/download/memcached-win64-1.4.4-14.zip](http://static.runoob.com/download/memcached-win64-1.4.4-14.zip)
* 32位系统 1.4.5版本：
  [http://static.runoob.com/download/memcached-1.4.5-x86.zip](http://static.runoob.com/download/memcached-1.4.5-x86.zip)
* 64位系统 1.4.5版本：
  [http://static.runoob.com/download/memcached-1.4.5-amd64.zip](http://static.runoob.com/download/memcached-1.4.5-amd64.zip)

在 1.4.5 版本以前 memcached 可以作为一个服务安装，而在 1.4.5 及之后的版本删除了该功能。因此我们以下介绍两个不同版本 1.4.4 及 1.4.5的不同安装方法：

---

## memcached &lt;1.4.5 版本安装

1、解压下载的安装包到指定目录。

2、在 1.4.5 版本以前 memcached 可以作为一个服务安装，使用管理员权限运行以下命令：

```
c:\memcached\memcached.exe -d install
```

**注意：**你需要使用真实的路径替代 c:\memcached\memcached.exe。

3、然后我们可以使用以下命令来启动和关闭 memcached 服务：

```
c:\memcached\memcached.exe -d start
c:\memcached\memcached.exe -d stop
```

4、如果要修改 memcached 的配置项, 可以在命令行中执行_regedit.exe_命令打开注册表并找到 "_HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Services\memcached_" 来进行修改。

如果要提供 memcached 使用的缓存配置 可以修改_ImagePath_为:

```
"c:\memcached\memcached.exe" -d runservice -m 512
```

**-m 512**意思是设置 memcached 最大的缓存配置为512M。

此外我们还可以通过使用 "_c:\memcached\memcached.exe -h_" 命令查看更多的参数配置。

5、如果我们需要卸载 memcached ，可以使用以下命令：

```
c:\memcached\memcached.exe -d uninstall
```

---

## memcached &gt;= 1.4.5 版本安装

1、解压下载的安装包到指定目录。

2、在 memcached1.4.5 版本之后，memcached 不能作为服务来运行，需要使用任务计划中来开启一个普通的进程，在 window 启动时设置 memcached自动执行。

我们使用管理员身份执行以下命令将 memcached 添加来任务计划表中：

```
schtasks /create /sc onstart /tn memcached /tr "'c:\memcached\memcached.exe' -m 512"
```

**注意：**你需要使用真实的路径替代 c:\memcached\memcached.exe。

**注意：-m 512**意思是设置 memcached 最大的缓存配置为512M。

**注意：**我们可以通过使用 "_c:\memcached\memcached.exe -h_" 命令查看更多的参数配置。

3、如果需要删除 memcached 的任务计划可以执行以下命令：

```
schtasks /delete /tn memcached
```



