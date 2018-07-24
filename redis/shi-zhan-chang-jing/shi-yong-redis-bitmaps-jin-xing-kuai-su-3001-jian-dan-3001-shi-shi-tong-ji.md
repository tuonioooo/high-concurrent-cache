# 使用Redis bitmaps进行快速、简单、实时统计

### 一、什么是BitSet？

注：以下内容来自JDK API:

BitSet类实现了一个按需增长的位向量。位Set的每一个组件都有一个boolean值。用非负的整数将BitSet的位编入索引。可以对每个编入索引的位进行测试、设置或者清除。通过逻辑与、逻辑或和逻辑异或操作，可以使用一个 BitSet修改另一个BitSet的内容。

默认情况下，set 中所有位的初始值都是false。

每个位 set 都有一个当前大小，也就是该位 set 当前所用空间的位数。注意，这个大小与位 set 的实现有关，所以它可能随实现的不同而更改。位 set 的长度与位 set 的逻辑长度有关，并且是与实现无关而定义的。

### 二、Java BitSet实现原理

在java中，BitSet的实现位于java.util包中：

```
public class BitSet implements Cloneable, java.io.Serializable 
{
    private final static int ADDRESS_BITS_PER_WORD = 6;
    private final static int BITS_PER_WORD = 1 << ADDRESS_BITS_PER_WORD;
    private final static int BIT_INDEX_MASK = BITS_PER_WORD - 1;

    /* Used to shift left or right for a partial word mask */
    private static final long WORD_MASK = 0xffffffffffffffffL;

    private static final ObjectStreamField[] serialPersistentFields =
     {
        new ObjectStreamField("bits", long[].class),
    };

    /**
     * The internal field corresponding to the serialField "bits".
     */
    private long[] words;

    .....
}
```

可以看到，BitSet的底层实现是使用long数组作为内部存储结构的，所以BitSet的大小为long类型大小\(64位\)的整数倍。

它有两个构造函数：

1、BitSet\(\)：创建一个新的位 set，默认大小是64位。

```
public BitSet() 
{
        initWords(BITS_PER_WORD);
        sizeIsSticky = false;
}

```

　2、BitSet\(int nbits\)：创建一个位set，它的初始大小足以显式表示索引范围在 0 到 nbits-1 的位。

