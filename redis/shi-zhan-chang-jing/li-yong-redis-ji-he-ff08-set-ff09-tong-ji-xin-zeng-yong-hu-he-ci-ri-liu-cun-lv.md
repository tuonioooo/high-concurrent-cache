# 利用Redis集合（Set）统计新增用户和次日留存率

**新增用户：**在某个时间段（一般为一整天）新登录应用的用户数，一般通过用户设备号判断用户是否是第一次登录应用

**次日留存率**：（当天新增的用户中，在注册的第2天还登录的用户数）/第一天新增总用户数

公司有专门的统计数据库，但是每张表的数据量都非常大，都是百万、千万级别，如果用常规的SQL语句去查找，查找的速度会非常慢，很不现实。结合实际的业务，最后想到用Redis的集合去计算处理数据。

**1、**首先我们需要计算所有出现过的用户设备号\(SELECT distinct\(user\_device\) FROM \`statistic\_20170801\` UNION SELECT distinct\(user\_device\) FROM \`statistic\_20170802\`\)，把这些设备号存在redis的集合中**\($redis-&gt;sadd\('user\_device','6ab3d84ace678644f44645'\)\)**；

---

**2、**再计算当天出现的所有设备号\(SELECT distinct\(user\_device\) FROM \`statistic\_20170803\`\)，将这些设备号存在redis临时集合中**\($redis-&gt;sadd\('tmp\_20170803','5a6vhbb84ace678fgv44f44645'\)**；

---

**3、**利用redis的**sdiffstore**命令，返回**user\_device**和**tmp\_20170803**两个集合的差集**\($redis-&gt;sdiffstore\('new\_20170803',"user\_device**","**tmp\_20170803**\)\)，这个新的**new\_20170803**集合，就是当日\(20170803\)的新增用户，为避免占用内存，需给这个**new\_20170803**集合设置过期时间，由于要算次日留存率，所以**new\_20170803**集合的生存期设为1天\(同理，要算七日留存率，则设置7天生存期\)；

---

**4、**利用redis的**SCARD**命令，得出**new\_20170803**这个集合的元素数量**\($redis-&gt;SCARD\('new\_20170803**'\)\)，就是20170803新增的用户数量；

---

**5、**不要忘记，将当天的用户设备号合并进**user\_device**集合中，利用redis的**SUNIONSTORE**命令，**$redis-&gt;sunionstore**\('**user\_device**',"**user\_device**","**tmp\_20170803**"\),再用**$redis-&gt;del\('tmp\_20170803**'\),删除这个临时集合；

---

**6**、计算次日留存率，我们要计算8月4日出现的所有设备号\(SELECT distinct\(user\_device\) FROM \`statistic\_20170804\`\)，将这些设备号存在redis临时集合中**\($redis-&gt;sadd\('tmp\_20170804','5a6vhbb84ace678fgv44f44645'\)**；

**7、**利用redis的**SINTERSTORE**命令，**$redis-&gt;sinterstore\('next\_day\_ retention',"new\_20170803**","**tmp\_20170804**"\)，返回**new\_20170803**和**tmp\_20170804**两个集合的并集，就是8月3日新增的用户，8月4日也正常登陆的用户集合了；

---

**8、**利用redis的**SCARD**命令，得出**next\_day\_retention**这个集合的元素数量**\($redis-&gt;SCARD\('next\_day\_retention'\)\)**，就是当天新增的用户，在注册的第2天还登录的用户数；

---

**9、**之前给**new\_20170803**这个集合设置一天的生存期就派上用场了，利用redis的**SCARD**命令，得出**new\_20170803**这个集合的元素数量\(**$redis-&gt;SCARD\('new\_20170803**'\)\)，就是20170803新增的用户数量；

---

**10、**再通过除法运算，**$redis-&gt;SCARD\('next\_day\_retention'\) / $redis-&gt;SCARD\('new\_20170803**'\)，得出的就是8月3日的次日留存率了。

