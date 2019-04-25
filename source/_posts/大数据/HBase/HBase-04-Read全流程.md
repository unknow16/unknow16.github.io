---
title: HBase-04-Read全流程
date: 2018-11-03 17:33:00
tags: HBase
---

#### 本文思路
1. 介绍HBase的两种读取模式：Get与Scan
2. Client如何发送请求到对应的RegionServer
3. RegionServer侧如何处理一次读取请求
4. 本文内容总结，并列出了关于Scan流程的更多细节问题

## Get读取模式
Get是指基于确切的RowKey去获取一行数据，通常被称之为随机点查，这正是HBase所擅长的读取模式。一次Get操作，包含两个主要步骤：

#### 1. 构建Get

基于RowKey构建Get对象的最简单示例代码如下：
```
final byte[] key = Bytes.toBytes("66660000431^201803011300");
Get get = new Get(key);
```
可以为构建的Get对象指定返回的列族：
```
final byte[] family = Bytes.toBytes("I");
// 指定返回列族I的所有列
get.addFamily(family);
```
也可以直接指定返回某列族中的指定列：
```
final byte[] family = Bytes.toBytes("I");
final byte[] qualifierMobile = Bytes.toBytes("M");
// 指定返回列族I中的列M
get.addColumn(family, qualifierMobile);
```

#### 2. 发送Get请求并且获取对应的记录

与写数据类似，发送Get请求的接口也是由Table提供的，获取到的一行记录，被封装成一个Result对象。也可以这么理解一个Result对象：

- 关联一行数据，一定不可能包含跨行的结果
- 包含一个或多个被请求的列(可能包含所有列，也可能仅包含部分列)

示例代码如下：
```
try (Table table = conn.getTable(TABLE)) {
  final byte[] key = Bytes.toBytes("66660000431^201803011300");
  Get get = new Get(key);
  // 记录Table提供的get接口获取一行记录
  Result result = table.get(get);
  // 通过CellScanner来遍历该Result中的所有列
  CellScanner scanner = result.cellScanner();
  while (scanner.advance()) {
     Cell cell = scanner.current();
     // 读取Cell中的信息...
  }    
}
```
上面给出的是一次随机获取一行记录的例子，但事实上，一次获取多行记录的需求也是普遍存在的，Table中也定义了Batch Get的接口，这样可以在一次网络请求中同时获取多行数据。示例代码如下：
```
try (Table table = conn.getTable(TN)) {
  // 基于多个Get对象构建一个List
  List<Get> gets = new ArrayList<>(8);
  gets.add(new Get(Bytes.toBytes("11110000431^201803011300")));
  gets.add(new Get(Bytes.toBytes("22220000431^201803011300")));
  gets.add(new Get(Bytes.toBytes("33330000431^201803011300")));
  gets.add(new Get(Bytes.toBytes("44440000431^201803011300")));
  gets.add(new Get(Bytes.toBytes("55550000431^201803011300")));
  gets.add(new Get(Bytes.toBytes("66660000431^201803011300")));
  gets.add(new Get(Bytes.toBytes("77770000431^201803011300")));
  gets.add(new Get(Bytes.toBytes("88880000431^201803011300")));
  // 调用Table的Batch Get的接口获取多行记录
  Result[] results = table.get(gets);
  for (Result result : results) {
    CellScanner scanner = result.cellScanner();
      while (scanner.advance()) {
        Cell cell = scanner.current();
          // 读取Cell中的信息...
      }
  }
}
```
关于Batch Get需要补充说明一点信息：获取到的Result列表中的结果的顺序，与给定的RowKey顺序是一致的。

## Scan
HBase中的数据表通过划分成一个个的Region来实现数据的分片，每一个Region关联一个RowKey的范围区间，而每一个Region中的数据，按RowKey的字典顺序进行组织。

正是基于这种设计，使得HBase能够轻松应对这类查询："指定一个RowKey的范围区间，获取该区间的所有记录"， 这类查询在HBase被称之为Scan。

一次Scan操作，包括如下几个关键步骤：

#### 1. 构建Scan

最简单也最常用的构建Scan对象的方法，就是仅仅指定Scan的StartRow与StopRow。示例如下：
```
final byte[] startKey = Bytes.toBytes("66660000431^20180301");
final byte[] stopKey = Bytes.toBytes("66660000431^20180501");
Scan scan = new Scan();
/**
  * 2.0版本之前，设置Scan Key Range的接口为：
  * scan.setStartRow(startKey).setStopRow(stopKey);
  * 但在2.0版本中，该接口已经被标注为Deprecated接口，即已不推荐使用.
  * 下面是推荐的接口：
  */
scan.withStartRow(startKey).withStopRow(stopKey);
```
如果StartRow未指定，则本次Scan将从表的第一行数据开始读取。

如果StopRow未指定，而且在不主动停止本次Scan操作的前提下，本次Scan将会一直读取到表的最后一行记录。

如果StartRow与StopRow都未指定，那本次Scan就是一次全表扫描操作。

同Get类似，Scan也可以主动指定返回的列族或列:
```
final byte[] family = Bytes.toBytes("I");
/**
  * 为本次Scan操作指定返回的列族
  * scan.addFamily(family);
  */ 
final byte[] qualifierMobile = Bytes.toBytes("M");
scan.addColumn(family, qualifierMobile);
```

#### 2.获取ResultScanner
```
ResultScanner scanner = table.getScanner(scan);
```
#### 3.遍历查询结果
```
Result result = null;
// 通过scanner.next方法获取返回的每一行数据
while ((result = scanner.next()) != null) {
    // 读取result中的结果...
}
```
#### 4.关闭ResultScanner
```
scanner.close();
```

如果基于Java传统的try-catch-finally语法，上述close方式需要在finally模块显式调用。但如果是是基于try-with-resource语法，则由Java框架自动调用。

将上面1~4步骤联合起来的示例代码如下：
```
try (Table table = conn.getTable(TABLE)) {
  final byte[] start = Bytes.toBytes("66660000431^20180301");
  final byte[] stop = Bytes.toBytes("66660000431^20180501");
  Scan scan = new Scan();
  scan.withStartRow(start).withStopRow(stop);
  // 基于try-with-resource语法，try语句块结束后
  // scanner的close方法会自动被调用
  try (ResultScanner scanner = table.getScanner(scan)) {
    Result result = null;
    while ((result = scanner.next()) != null) {
      // 读取result中的结果...
    }
  }
}
```

## Scan的其它重要参数
1. Caching: 设置一次RPC请求批量读取的Results数量

下面的示例代码设定了一次读取回来的Results数量为100：


```
scan.setCaching(100);
```


Client每一次往RegionServer发送scan请求，都会批量拿回一批数据(由Caching决定过了每一次拿回的Results数量)，然后放到本次的Result Cache中：

![image](https://note.youdao.com/yws/api/personal/file/3CB0C6F22877469CB47846F2836D5C18?method=download&shareKey=2bc3b45b42ac1ce50a6f50c6dfbf5aa6)

应用每一次读取数据时，都是从本地的Result Cache中获取的。如果Result Cache中的数据读完了，则Client会再次往RegionServer发送scan请求获取更多的数据。

2. Batch: 设置每一个Result中的列的数量

下面的示例代码设定了每一个Result中的列的数量的限制值为3：


```
scan.setBatch(3);
```


该参数适用于一行数据过大的场景，这样，一行数据被请求的列会被拆成多个Results返回给Client。

举例说明如下：

假设一行数据中共有十个列：


```
{Col01，Col02，Col03，Col04，Col05，Col06，Col07，Col08，Col09, Col10}
```


假设Scan中设置的Batch为3，那么，这一行数据将会被拆成4个Results返回：


```
Result1 -> {Col01，Col02，Col03}
Result2 -> {Col04，Col05，Col06}
Result3 -> {Col07，Col08，Col09}
Result4 -> {Col10}
```

关于Caching参数，我们说明了是Client每一次从RegionServer侧获取到的Results的数量，上例中，一行数据被拆成了4个Results，这将会导致Caching中的计数器被减了4次。结合Caching与Batch，我们再列举一个稍复杂的例子：

假设，Scan的参数设置如下：


```
final byte[] start = Bytes.toBytes("Row1");
final byte[] stop = Bytes.toBytes("Row5");
Scan scan = new Scan();
scan.withStartRow(start).withStopRow(stop);
scan.setCaching(10);
scan.setBatch(3);
```


待读取的数据RowKey与所关联的列集如下所示：



```
Row1:  {Col01，Col02，Col03，Col04，Col05，Col06，Col07，Col08，Col09，Col10}   
Row2:  {Col01，Col02，Col03，Col04，Col05，Col06，Col07，Col08，Col09，Col10，Col11}  
Row3:  {Col01，Col02，Col03，Col04，Col05，Col06，Col07，Col08，Col09，Col10}
```


再回顾一下Caching与Batch的定义：

- Caching:  影响一次读取返回的Results数量。
- Batch:  限定了一个Result中所包含的列的数量，如果一行数据被请求的列的数量超出Batch限制，那么这行数据会被拆成多个Results。

那么， Client往RegionServer第一次请求所返回的结果集如下所示：


```
Result1   ->   Row1:  {Col01，Col02，Col03}
Result2   ->   Row1:  {Col04，Col05，Col06}
Result3   ->   Row1:  {Col07，Col08，Col09}
Result4   ->   Row1:  {Col10}
Result5   ->   Row2:  {Col01，Col02，Col03}
Result6   ->   Row2:  {Col04，Col05，Col06}
Result7   ->   Row2:  {Col07，Col08，Col09}
Result8   ->   Row2:  {Col10，Col11}
Result9   ->   Row3:  {Col01，Col02，Col03}
Result10  ->   Row3:  {Col04，Col05，Col06}
```
3. Limit: 限制一次Scan操作所获取的行的数量

同SQL语法中的limit子句，限制一次Scan操作所获取的行的总量：

```
scan.setLimit(10000);
```

4. CacheBlock: RegionServer侧是否要缓存本次Scan所涉及的HFileBlocks

```
scan.setCacheBlocks(true);
```
5. Raw Scan:  是否可以读取到删除标识以及被删除但尚未被清理的数据


```
scan.setRaw(true);
```

6. MaxResultSize:  从内存占用量的维度限制一次Scan的返回结果集


```
#下面的示例代码将返回结果集的最大值设置为5MB：
scan.setMaxResultSize(5 * 1024 * 1024);
```

7.  Reversed Scan: 反向扫描

普通的Scan操作是按照字典顺序从小到大的顺序读取的，而Reversed Scan则恰好相反：

```
scan.setReversed(true);
```

8. 带Filter的Scan

Filter可以在Scan的结果集基础之上，对返回的记录设置更多条件值，这些条件可以与RowKey有关，可以与列名有关，也可以与列值有关，还可以将多个Filter条件组合在一起，等等。

最常用的Filter是SingleColumnValueFilter，基于它，可以实现如下类似的查询：

"返回满足条件{列I:D的值大于等于10}的所有行"

示例代码如下：
```
Filter filter = new SingleColumnValueFilter(FAMILY,
      QUALIFIER, CompareOperator.GREATER_OR_EQUAL,
      Bytes.toBytes("10"));
scan.setFilter(filter);
```
Filter丰富了HBase的查询能力，但使用Filter之前，需要注意一点：Filter可能会导致查询响应时延变的不可控制。因为我们无法预测，为了找到一条符合条件的记录，背后需要扫描多少数据量，如果在有效限制了Scan范围区间(通过设置StartRow与StopRow限制)的前提下，该问题能够得到有效的控制。这些信息都要求使用Filter之前应该详细调研自己的业务数据模型。

## Client发送读取请求到RegionServer
无论是Get，还是Scan，Client在发送请求到RegionServer之前，也需要先获取路由信息：

1. 定位该请求所关联的Region

因为Get请求仅关联一个RowKey，所以，直接定位该RowKey所关联的Region即可。对于Scan请求，先定位Scan的StartRow所关联的Region。

2. 往RegionServer发送读取请求

该过程与《 Write全流程》的"数据路由"章节所描述的流程类似，不再赘述。

如果一次Scan涉及到跨Region的读取，读完一个Region的数据以后，需要继续读取下一个Region的数据，这需要在Client侧不断记录和刷新Scan的进展信息。如果一个Region中已无更多的数据，在scan请求的响应结果中会带有提示信息，这样可以让Client侧切换到下一个Region继续读取。

## RegionServer如何处理读取请求

#### 关于Read的命题

通过前面的文章我们已经了解了如下信息：

1. 一个表可能包含一个或多个Region

将HBase中拥有数亿行的一个大表，横向切割成一个个"子表"，这一个个"子表"就是Region。

![image](https://note.youdao.com/yws/api/personal/file/31FA240B1F6E4AE1A38F3E2E5B95AF2D?method=download&shareKey=6ddfc6e610e7a02990bcc0fd6d932c45)

2. 每一个Region中关联一个或多个列族

如果将Region看成是一个表的横向切割，那么，一个Region中的数据列的纵向切割，称之为一个Column Family。每一个列，都必须归属于一个Column Family，这个归属关系是在写数据时指定的，而不是建表时预先定义。

![image](https://note.youdao.com/yws/api/personal/file/E8672BC14E5F4E949C00EFA0C384DD49?method=download&shareKey=e7290eda1e03b41cbf2475669fcc0e80)

3. 每一个列族关联一个MemStore，以及一个或多个HFiles文件

上面的关于“Region与多列族”的图中，泛化了Column Family的内部结构。下图是包含MemStore与HFile的Column Family组成结构：

![image](https://note.youdao.com/yws/api/personal/file/A507F1EEFBB746BFA4861154125DA7E5?method=download&shareKey=96cdbd5b84d90ab59722078b4979d6d5)

HFile数据文件存在于底层的HDFS中，上图中只是为了方便阐述HFile与Column Family之间的关系。

在HBase的源码实现中，将一个Column Family抽象成一个Store对象。可以这么简单理解Column Family与Store的概念差异：Column Family更多的是面向用户层可感知的逻辑概念，而Store则是源码实现中的概念，是关于一个Column Family的抽象。

4. 每一个MemStore中可能涉及一个Active Segment，以及一个或多个Immutable Segments

![image](https://note.youdao.com/yws/api/personal/file/6AE48A06AE31471BA23DC49F5AFDC98B?method=download&shareKey=3a6eb3f7b981050ce0486f9b66595586)

扩展到一个Region包含两个Column Family的情形：

![image](https://note.youdao.com/yws/api/personal/file/37D137E78BBC4C31B4F021FCE1FEBF2B?method=download&shareKey=e49f6e8f2dd28c7c3e27c37c7e714174)

5. HFile由Block构成，默认地，用户数据被按序组织成一个个64KB的Block

HFile V1的结构虽已过时，但非常有助于你理解HFile的核心设计思想：

![image](https://note.youdao.com/yws/api/personal/file/A24A96FE6B984EE5B02E9B775CD8D932?method=download&shareKey=351313350ec4ee37fa2d2f4745c89851)

- Data Block(上图中左侧的Data块)：保存了实际的KeyValue数据。
- Data Index：关于Data Block的索引信息。

HFile V2只不过在HFile V1基础上做的演进，将Data Index信息以及BloomFilter的数据也分成了多层。

当前阶段，你只需要了解到：基于一个给定的RowKey，HFile中提供的索引信息能够快速查询到对应的Data Block。

在重新温习了上述内容以后，我们也大致了解了关于HBase读取我们所面临的问题是什么。关于HBase Read的命题可以定义为：如何从1个或多个列族(1个或多个MemStore Segments+1个或多个HFiles)所构成的Region中读取用户所期望的数据？这些数据默认必须是未被标记删除的、未过期的而且是最新版本的数据。

#### 将Get看作一类特殊的Scan
无论是读取一行数据，还是读取指定RowKey范围的读取一系列数据，所面临的问题其实是类似的，因此，可以将Get看作是一种特殊的Scan，只不过它的StartRow与StopRow重叠，事实上，RegionServer侧处理Get请求时的确先将Get先转换成了一个Scan操作。

#### 合理组织所有的KeyValue数据源
在Store/Column Family内部，KeyValue可能存在于MemStore的Segment中，也可能存在于HFile文件中，无论是Segment还是HFile，我们统称为KeyValue数据源。

在本文的第一部分介绍如何执行Scan操作时，我们讲到了Client侧使用一个ResultScanner来抽象地描述一次Scan操作，ResultScanner屏蔽掉了往RegionServer发送请求以及一个Region读取完成以后切换到下一个Region等细节信息。

初次阅读RegionServer/Region的读取流程所涉及的源码时，会被各色各样的Scanner类整的晕头转向，HBase使用了各种Scanner来抽象每一层/每一类KeyValue数据源的Scan操作：


- 关于一个Region的读取，被封装成一个RegionScanner对象。
- 每一个Store/Column Family的读取操作，被封装在一个StoreScanner对象中。
- SegmentScanner与StoreFileScanner分别用来描述关于MemStore中的Segment以及HFile的读取操作。
- StoreFileScanner中关于HFile的实际读取操作，由HFileScanner完成。

RegionScanner的构成如下图所示：

![image](https://note.youdao.com/yws/api/personal/file/99B9F01FFB5B4B09912FC3D88E1E32CF?method=download&shareKey=a080ee7ca576d54a293d5f7a86949f46)

在StoreScanner内部，多个SegmentScanner与多个StoreFileScanner被组织在一个称之为KeyValueHeap的对象中：

![image](https://note.youdao.com/yws/api/personal/file/594B7C8E9EF74463808AD13EA101E9D5?method=download&shareKey=c1012926da1fcc744122a4d745cfafb4)

每一个Scanner内部有一个指针指向当前要读取的KeyValue，KeyValueHeap的核心是一个优先级队列(PriorityQueue)，在这个PriorityQueue中，按照每一个Scanner当前指针所指向的KeyValue进行排序：


```
// 用来组织所有的Scanner
protected PriorityQueue<KeyValueScanner> heap = null;

// PriorityQueue当前排在最前面的Scanner
protected KeyValueScanner current = null;
```

同样的，RegionScanner中的多个StoreScanner，也被组织在一个KeyValueHeap对象中：

![image](https://note.youdao.com/yws/api/personal/file/0C8CB1539EC44EE5B9EB9FC314D56F08?method=download&shareKey=a8a7211d3f0ff66e1af2eb50d0bc3f1e)

#### 读取KeyValue的基础Scanner接口：KeyValueScanner
KeyValueScanner定义了读取KeyValue的基础接口：
```
/**
  * 查看当前Scanner中当前指针位置的KeyValue，该接口不会移动指针.
  */
Cell peek();

/**
  * 返回Scanner当前指针位置的KeyValue，而后移动指针到下一个KeyValue.
  */
Cell next() throws IOException;

/**
  * 将当前Scanner的指针定位到指定的KeyValue的位置,如果不存在，则定位到
  * 比该Cell大的下一个KeyValue位置.Seek操作会从当前的HFile Block的开
  * 始位置查找.
  */
boolean seek(Cell key) throws IOException;

/**
  * 与seek接口类似，也是将当前Scanner的指针定位到指定的KeyValue的位置,
  * 如果不存在，则定位到比该KeyValue大的下一个KeyValue位置.与seek接口不
  * 同点在于，该操作会从上一次读到的HFile Block的位置开始查找.
  */
boolean reseek(Cell key) throws IOException;

/**
  * 与seek/reseek类似，但不同点在于采用了Lazy Seek机制来降低磁盘IO请求，
  * 其原理在于充分利用Bloom Filter的判断结果，以及待Seek的KeyValue与该
  * Scanner中最大时间戳的对比，减少一些不必要的Seek操作
  */
boolean requestSeek(Cell kv, boolean forward, 
     boolean useBloom) throws IOException;
```
实现了KeyValueScanner接口类的主要Scanner包括：

- StoreFileScanner
- SegmentScanner
- StoreScanner

## RegionScanner初始化
RegionScanner初始化过程，包括几个关键操作：

1. 获取ReadPoint

ReadPoint决定了此次Scan操作能看到哪些数据。Scan过程中新写入的数据，对此次Scan是不可见的。

2. 按需选择对应的Store，并初始化对应的StoreScanner

StoreScanner在初始化的时候，也会按需选择对应的SegmentScanner以及StoreFileScanner，筛选规则包括：


- 如果一次Scan操作指定了Time Range，则只选择与该Time Range有关的Scanners。
- 对于Get操作，可以通过BloomFilter过滤掉不符合条件的Scanners。


StoreScanner中筛选出了Scanner以后，会将每一个Scanner seek到Scan的StartRow位置：

![image](https://note.youdao.com/yws/api/personal/file/9C8B3DDA781A41CDAC5B2ADA5DFDEA0E?method=download&shareKey=758492e284d27498ec664b73662b8ef5)

## 通过next请求读取一个个KeyValue
如果将RegionScanner理解成一个内部构造复杂的机器，而驱动这个机器运转的动力源自Client侧的一次次scan请求，scan请求通过调用RegionScanner的next方法来获取一个个KeyValue。

为了简单的解释该流程，我们先假定一个RegionScanner中仅包含一个StoreScanner，那么，这个RegionScanner中的核心读取操作，是由StoreScanner完成的，我们进一步假定StoreScanner由4个Scanners组成（我们泛化了SegmentScanner与StoreFileScanner的区别，统称为Scanner），直观起见，在下图中我们使用了四种不同的颜色（ScannerA~ScannerD为随机名称，请忽略它们在名称上的顺序）：


![image](https://note.youdao.com/yws/api/personal/file/9C0981B648C74AF3AC579F96B8E1AA69?method=download&shareKey=5bb358fc72fadcc09df765cfff1882b1)

每一个Scanner中都有一个current指针指向下一个即将要读取的KeyValue，KeyValueHeap中的PriorityQueue正是按照每一个Scanner的current所指向的KeyValue进行排序。

第一次next请求，将会返回ScannerA中的Row01:FamA:Col1，而后ScannerA的指针移动到下一个KeyValue Row01:FamA:Col2，PriorityQueue中的Scanners排序依然不变：

![image](https://note.youdao.com/yws/api/personal/file/B390F1E1B23F445B9D9E1CC044FBF1B7?method=download&shareKey=1d058488b2191de61c3c4a8f2be7e565)

第二次next请求，依然返回ScannerA中的Row01:FamA:Col2，ScannerA的指针移动到下一个KeyValue Row02:FamA:Col1，此时，PriorityQueue中的Scanners排序发生了变化：

![image](https://note.youdao.com/yws/api/personal/file/B08A19A746AB4B39A322B407C8544F9E?method=download&shareKey=9b777e44b8fc6b4ae208a6606b7530fe)

下一次next请求，将会返回ScannerB中的KeyValue.....周而复始，直到某一个Scanner所读取的数据耗尽，该Scanner将会被close，不再出现在上面的PriorityQueue中。

SegmentScanner/StoreFileScanner中返回的KeyValue，包含了各种类型的KeyValue：

- 已被更新过的旧KeyValue
- 已被标记删除但尚未被及时清理的KeyValue
- 已过期的尚未被及时清理的KeyValue
- 用来描述一次删除操作的KeyValue(删除还包含了多种类型)
- 承载最新用户数据的普通KeyValue

因此，在StoreScanner层，需要对这些KeyValue做更复杂的逻辑校验，这些校验由ScanQueryMatcher完成。默认地，可作为返回数据的KeyValue，应该满足如下条件：

- KeyValue类型为Put
- KeyValue所关联的列为用户Scan所涉及的列
- KeyValue的时间戳符合Scan的TimeRange要求
- 版本最新
- 未被标记删除
- 通过了Filter的过滤条件

上述条件，只针对一些普通的Scan，不同的Scan参数配置，可能会导致条件集发生变化，如Scan启用了Raw Scan模式时，Delete类型的KeyValue也会被返回。另外，上面的这些条件所罗列的顺序，也未遵循实际的检查顺序，而实际的检查顺序也是严格的，如果颠倒就可能会导致Bug。

在Scanner中，如果允许读取多个版本（由Scan#readVersions配置），那正常的读取顺序应该为：

![image](https://note.youdao.com/yws/api/personal/file/11ED46ECB4A44F188BA91BF3E78E510C?method=download&shareKey=8c38a1ba30b6a0e617e91d6aa3ee00c4)

上面这种读取的顺序与实际存在的数据的逻辑顺序也是相同的。

由于不同的Scan所读取的每一行中的数据不同，有的限定了列的数量，有的限定了版本的数量，这使得读取时可以通过一些优化，减少不必要的数据扫描。如某次Scan在允许读多个版本的同时，限定了只读取C1~C3，那么，读取顺序应该为：

![image](https://note.youdao.com/yws/api/personal/file/D9051F96CBD4490391F18A51E95A9E55?method=download&shareKey=a9a63b9de4229eaac33950be0890e1fa)

最普通的Scan，其实只需要读取每一列的最新版本即可，那读取的顺序应该为：

![image](https://note.youdao.com/yws/api/personal/file/F0202D9DBF1148B481ECBDB58D75D9B9?method=download&shareKey=b7a522d6d64bcb31aac22285bc2302b8)

通过上面几张图，我们其实是想说明在Scanner内部需要具备这样的一些基础能力：

- 如果只需要当前列的最新版本，那么Scanner应该可以跳过当前列的其它版本，而且将指针移到下一列的开始位置。
- 如果当前行的所要读取的列都已读完，那么，Scanner应该可以跳过该行剩余的列，将指针移动到下一行的开始位置。


我们知道KeyValueScanner定义了基础的seek/reseek/requestSeek等接口，可以将指针移动到指定KeyValue位置。但关于指针如何移动的决策信息，由谁来提供？

这些信息也是由ScanQueryMatcher提供的。ScanQueryMatcher对每一个KeyValue的逻辑检查结果称之为MatchCode，MatchCode不仅包含了是否应该返回该KeyValue的结果，还可能给出了Scanner的下一步操作的提示信息。关于它的枚举值，简单举例如下：



```
# 包含当前KeyValue，并提示Scanner当前行已无需继续读取，请Seek到下一行。
INCLUDE_AND_SEEK_NEXT_ROW

# 包含当前KeyValue，并提示Scanner当前列已无需继续读取，请Seek到下一列。
INCLUDE_AND_SEEK_NEXT_COL
```

无论是StoreScanner还是RegionScanner，返回的都是符合条件的KeyValue列表。这些KeyValues在RSRpcServices层被进一步组装成Results响应给Client侧。

## 总结
Scan涉及了太多的细节内容，本文只粗略介绍了Scan的一些核心思路，这与本系列文章最初的定位有关，当然也受限于本文的篇幅。 本文主要介绍了如下内容：


- 介绍HBase的两种读取模式：Get与Scan

1. Client如何发起一次Get请求，Get的关键参数
2. Client如何发起一次Scan请求，Scan的关键参数


- 重点介绍了RegionServer侧关于Scan的处理流程：

1. 如何用Scanner来抽象描述关于Region的读取操作
2. 关于读取KeyValue的基础Scanner接口定义 
3. RegionScanner初始化时的关键操作
4. Client侧的一次次scan请求如何驱动RegionScanner内部的读取操作
5. 从StoreFileScanner/SegmentScanner中读取出来的原始KeyValue如何被合理的校验
6. Scanner读取时如何跳过一些不必要的数据
