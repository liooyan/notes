# 1 PagedBytes

用于存储 byte数组 的容器。

通过 byte 二维数组，存储数据。当数据需要扩充是，只需要扩展byte的一维数组，减少数据复制。



# 2 主要方法



## 2.1 构造函数

```java
  public PagedBytes(int blockBits) {
    assert blockBits > 0 && blockBits <= 31 : blockBits;
    this.blockSize = 1 << blockBits;
    this.blockBits = blockBits;
    blockMask = blockSize-1;
    upto = blockSize;
    bytesUsedPerBlock = RamUsageEstimator.alignObjectSize(blockSize + RamUsageEstimator.NUM_BYTES_ARRAY_HEADER);
    numBlocks = 0;
  }
```

通过 参数 blockBits 确定 byte 每个元素的长度。主要cheng'yu





## 2.2 addBlock

内部调用，当

