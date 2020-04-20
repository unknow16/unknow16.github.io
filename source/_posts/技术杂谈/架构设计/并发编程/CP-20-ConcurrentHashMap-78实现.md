---
title: CP-20-ConcurrentHashMap-78实现
date: 2018-11-13 21:36:58
tags: ConcurrentPrograming
---

本文目录
- Java7 ConcurrentHashMap
- Java8 ConcurrentHashMap

## Java7 ConcurrentHashMap

采用分段锁+数组+链表的数据结构

#### 线程不安全的HashMap
因为多线程环境下，使用Hashmap进行put操作会引起死循环，导致CPU利用率接近100%，所以在并发情况下不能使用HashMap。

#### 效率低下的HashTable容器
HashTable容器使用synchronized来保证线程安全，但在线程竞争激烈的情况下HashTable的效率非常低下。因为当一个线程访问HashTable的同步方法时，其他线程访问HashTable的同步方法时，可能会进入阻塞或轮询状态。如线程1使用put进行添加元素，线程2不但不能使用put方法添加元素，并且也不能使用get方法来获取元素，所以竞争越激烈效率越低。
    
#### 锁分段技术
HashTable容器在竞争激烈的并发环境下表现出效率低下的原因，是因为所有访问HashTable的线程都必须竞争同一把锁，那假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHashMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。有些方法需要跨段，比如size()和containsValue()，它们可能需要锁定整个表而而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放所有段的锁。这里“按顺序”是很重要的，否则极有可能出现死锁，因此在ConcurrentHashMap内部，段数组是final的，并且其成员变量实际上也是final的，但是，仅仅是将数组声明为final的并不保证数组成员也是final的，这需要实现上的保证。这可以确保不会出现死锁，因为获得锁的顺序是固定的。

ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。Segment是一种可重入锁ReentrantLock，在ConcurrentHashMap里扮演锁的角色，HashEntry则用于存储键值对数据。一个ConcurrentHashMap里包含一个Segment数组，Segment的结构和HashMap类似，是一种数组和链表结构， 一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素， 每个Segment守护者一个HashEntry数组里的元素,当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁。
    
![image](https://note.youdao.com/yws/api/personal/file/6819B9ED59074CD08CE53D16CDFDD46D?method=download&shareKey=fb7e195048f759607123d08011f27dae)


ConcurrentHashMap(1.7及之前)中主要实体类就是三个：ConcurrentHashMap（整个Hash表）,Segment（桶），HashEntry（节点），对应上面的图可以看出之间的关系

```
/** 
* The segments, each of which is a specialized hash table 
*/  
final Segment<K,V>[] segments;
```

#### 不变(Immutable)和易变(Volatile)
ConcurrentHashMap完全允许多个读操作并发进行，读操作并不需要加锁。如果使用传统的技术，如HashMap中的实现，如果允许可以在hash链的中间添加或删除元素，读操作不加锁将得到不一致的数据。ConcurrentHashMap实现技术是保证HashEntry几乎是不可变的。HashEntry代表每个hash链中的一个节点，其结构如下所示：

```
static final class HashEntry<K,V> {  
     final K key;  
     final int hash;  
     volatile V value;  
     final HashEntry<K,V> next;  
 }
```

可以看到除了value不是final的，其它值都是final的，这意味着不能从hash链的中间或尾部添加或删除节点，因为这需要修改next 引用值，所有的节点的修改只能从头部开始。对于put操作，可以一律添加到Hash链的头部。但是对于remove操作，可能需要从中间删除一个节点，这就需要将要删除节点的前面所有节点整个复制一遍，最后一个节点指向要删除结点的下一个结点。这在讲解删除操作时还会详述。为了确保读操作能够看到最新的值，将value设置成volatile，这避免了加锁。

#### 定位段和每个段中的槽
为了加快定位段以及每个段中hash槽的速度，每个段的hash槽的个数都是2^n，这使得通过位运算就可以定位段和段中hash槽的位置。当并发级别为默认值16时，也就是段的个数，hash值的高4位决定分配在哪个段中。但是我们也不要忘记《算法导论》给我们的教训：hash槽的的个数不应该是 2^n，这可能导致hash槽分配不均，这需要对hash值重新再hash一次。
```
// 根据key的 hash 值找到 Segment 数组中的位置 j
// 默认情况下segmentShift为28，segmentMask为15
 int j = (hash >>> segmentShift) & segmentMask;

```
既然ConcurrentHashMap使用分段锁Segment来保护不同段的数据，那么在插入和获取元素的时候，必须先通过哈希算法定位到Segment。可以看到ConcurrentHashMap会首先使用Wang/Jenkins hash的变种算法对元素的hashCode进行一次再哈希。

再哈希，其目的是为了减少哈希冲突，使元素能够均匀的分布在不同的Segment上，从而提高容器的存取效率。假如哈希的质量差到极点，那么所有的元素都在一个Segment中，不仅存取元素缓慢，分段锁也会失去意义。

#### 重要参数
concurrencyLevel：并行级别、并发数、Segment 数，怎么翻译不重要，理解它。默认是 16，也就是说 ConcurrentHashMap 有 16 个 Segments，所以理论上，这个时候，最多可以同时支持 16 个线程并发写，只要它们的操作分别分布在不同的 Segment 上。这个值可以在初始化的时候设置为其他值，但是一旦初始化以后，它是不可以扩容的。

initialCapacity：初始容量，这个值指的是整个 ConcurrentHashMap 的初始容量，实际操作的时候需要平均分给每个 Segment。

loadFactor：负载因子，之前我们说了，Segment 数组不可以扩容，所以这个负载因子是给每个 Segment 内部使用的。


#### 初始化
ConcurrentHashMap初始化时，计算出Segment数组的大小ssize和每个Segment中HashEntry数组的大小cap，并初始化Segment数组的第一个元素

其中ssize大小为2的幂次方，默认为16，cap大小也是2的幂次方，最小值为2，最终结果根据根据初始化容量initialCapacity进行计算

初始化完成，我们得到了一个 Segment 数组。我们就当是用 new ConcurrentHashMap() 无参构造函数进行初始化的，那么初始化完成后：

- Segment 数组长度为 16，不可以扩容
- Segment[i] 的默认大小为 2，负载因子是 0.75，得出初始阈值为 1.5，也就是以后插入第一个元素不会触发扩容，插入第二个会进行第一次扩容
- 这里初始化了 segment[0]，其他位置还是 null，至于为什么要初始化 segment[0]，后面的代码会介绍
- 当前 segmentShift 的值为 32 - 4 = 28，segmentMask 为 16 - 1 = 15，姑且把它们简单翻译为移位数和掩码，这两个值马上就会用到

#### put实现
当执行put方法插入数据时，根据key的hash值，在Segment数组中找到相应的位置，如果相应位置的Segment还未初始化，则通过CAS进行赋值，接着执行Segment对象的put方法通过加锁机制插入数据，实现如下：

场景：线程A和线程B同时执行相同Segment对象的put方法

1. 线程A执行tryLock()方法成功获取锁，则把HashEntry对象插入到相应的位置；
2. 线程B获取锁失败，则执行scanAndLockForPut()方法，在scanAndLockForPut方法中，会通过重复执行tryLock()方法尝试获取锁，在多处理器环境下，重复次数为64，单处理器重复次数为1，当执行tryLock()方法的次数超过上限时，则执行lock()方法挂起线程B；
3. 当线程A执行完插入操作时，会通过unlock()方法释放锁，接着唤醒线程B继续执行

#### get 过程
相对于 put 来说，get 真的不要太简单。

1. 计算 hash 值，找到 segment 数组中的具体位置，或我们前面用的“槽”
1. 槽中也是一个数组，根据 hash 找到数组中具体的位置
1. 到这里是链表了，顺着链表进行查找即可

#### size实现
因为ConcurrentHashMap是可以并发插入数据的，所以在准确计算元素时存在一定的难度，一般的思路是统计每个Segment对象中的元素个数，然后进行累加，但是这种方式计算出来的结果并不一样的准确的，因为在计算后面几个Segment的元素个数时，已经计算过的Segment同时可能有数据的插入或则删除，在1.7的实现中，采用了如下方式：

先采用不加锁的方式，连续计算元素的个数，最多计算3次：
1. 如果前后两次计算结果相同，则说明计算出来的元素个数是准确的；
2. 如果前后两次计算结果都不同，则给每个Segment进行加锁，再计算一次元素的个数；

#### putIfAbsent方法的使用以及返回值的含义
putIfAbsent方法主要是在向ConcurrentHashMap中添加键—值对的时候，它会先判断该键值对是否已经存在。

1. 如果不存在（新的entry），那么会向map中添加该键值对，并返回null。 
1. 如果已经存在，那么不会覆盖已有的值，直接返回已经存在的值。 

## Java8 ConcurrentHashMap 
1.8中放弃了Segment臃肿的设计，取而代之的是采用Node + CAS + Synchronized来保证并发安全进行实现，采用数组+链表+红黑树的数据结构

![image](https://note.youdao.com/yws/api/personal/file/CE5FA753A7BF438AA612E645727C4A4E?method=download&shareKey=f0240743a67a741ebeb65fde07324c9f)

只有在执行第一次put方法时才会调用initTable()初始化Node数组

#### 初始化

构造方法如下：
```
// 这构造函数里，什么都不干
public ConcurrentHashMap() {
}

public ConcurrentHashMap(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
               MAXIMUM_CAPACITY :
               tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
    this.sizeCtl = cap;
}

// 还有三个不常用的构造方法,省略
...
```
这个初始化方法有点意思，通过提供初始容量，计算了 sizeCtl，sizeCtl = 【 (1.5 * initialCapacity + 1)，然后向上取最近的 2 的 n 次方】。如 initialCapacity 为 10，那么得到 sizeCtl 为 16，如果 initialCapacity 为 11，11x1.5+1=17.5, 向上取2的n次方，得到 sizeCtl 为 32。

#### put过程
1. 得到key的hash值，然后进行死循环，其中有如下四个if语句块
2. 先判断table数组是否为空，为空则初始化，再次循环
3. 通过hash值找到数组table对应下标处的节点，然后判断是否为空，如果为空，则通过CAS放入即可，CAS成功然后put方法就算执行成功了，CAS失败那就是有并发操作，break进到下一个循环
4. 判断hash是否为MOVED值，是则帮助数据迁移
5. 最后的else, 到这里就是说，hash对应的数组table下标位置已经存在头节点f，而且不为空，用synchronized获取数组该位置的头结点的监视器锁，之后判断该位置是链表，还是红黑树，进行相应的插入或更新节点操作，之后会判断是否要将链表转换为红黑树或扩容数组，如果当前数组的长度小于 64，那么会选择进行数组扩容，之后再出现数组某下标处的链表节点数超过8个，会将其转换为红黑树。

#### get过程
1. 计算 hash 值
2. 根据 hash 值找到数组对应位置: (n - 1) & h
3. 根据该位置处结点性质进行相应查找
- 如果该位置为 null，那么直接返回 null 就可以了
- 如果该位置处的节点刚好就是我们需要的，返回该节点的值即可
- 如果该位置节点的 hash 值小于 0，说明正在扩容，或者是红黑树，后面我们再介绍 find 方法
- 如果以上 3 条都不满足，那就是链表，进行遍历比对即可

#### size实现
1.8中使用一个volatile类型的变量baseCount记录元素的个数，当插入新数据或则删除数据时，会通过addCount()方法更新baseCount

https://www.cnblogs.com/ITtangtang/p/3948786.html

https://www.jianshu.com/p/e694f1e868ec

https://javadoop.com/post/hashmap#toc3