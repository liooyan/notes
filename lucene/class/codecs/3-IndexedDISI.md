# 1 IndexedDISI

基于磁盘的文档id列表实现。

- 在 Lucene80DocValuesConsumer 中 DocsIDFileId   结构就通过这个类实现，将数据写入到文件中
- 同样通过IndexedDISI 可以加载 DocsIDFileId   的文档id遍历文档id



# 2 构造函数



```java

  IndexedDISI(IndexInput blockSlice, RandomAccessInput jumpTable, int jumpTableEntryCount, byte denseRankPower, long cost) throws IOException {
    if ((denseRankPower < 7 || denseRankPower > 15) && denseRankPower != -1) {
      throw new IllegalArgumentException("Acceptable values for denseRankPower are 7-15 (every 128-32768 docIDs). " +
          "The provided power was " + denseRankPower + " (every " + (int)Math.pow(2, denseRankPower) + " docIDs). ");
    }

    this.slice = blockSlice;
    this.jumpTable = jumpTable;
    this.jumpTableEntryCount = jumpTableEntryCount;
    this.denseRankPower = denseRankPower;
    final int rankIndexShift = denseRankPower-7;
    this.denseRankTable = denseRankPower == -1 ? null : new byte[DENSE_BLOCK_LONGS >> rankIndexShift];
    this.cost = cost;
  }
```

参数分别为：

- IndexInput blockSlice 文件中存储文档id的部分
- RandomAccessInput jumpTable  文件中存储文件id对应跳表部分
- jumpTableEntryCount 跳表的数量
- denseRankPower ：metaData中内容
- cost： 当前存储的文档数量



# 3 主要方法



这里主要关注几个方法

- nextDoc 迭代获取下一个文档id
- docID 获取当前的文档id
- index 获取当前文档id的编号，用于通过index获取对应的value值





