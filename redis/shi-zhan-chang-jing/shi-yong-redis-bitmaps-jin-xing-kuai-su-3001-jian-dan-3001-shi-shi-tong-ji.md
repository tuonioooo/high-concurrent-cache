# 使用Redis bitmaps进行快速、简单、实时统计

### 一、什么是BitSet？

　　注：以下内容来自JDK API:

　　BitSet类实现了一个按需增长的位向量。位Set的每一个组件都有一个boolean值。用非负的整数将BitSet的位编入索引。可以对每个编入索引的位进行测试、设置或者清除。通过逻辑与、逻辑或和逻辑异或操作，可以使用一个 BitSet修改另一个BitSet的内容。 

　　默认情况下，set 中所有位的初始值都是false。 

　　每个位 set 都有一个当前大小，也就是该位 set 当前所用空间的位数。注意，这个大小与位 set 的实现有关，所以它可能随实现的不同而更改。位 set 的长度与位 set 的逻辑长度有关，并且是与实现无关而定义的。 

[回到顶部](https://www.cnblogs.com/xujian2014/p/5491286.html#_labelTop)



