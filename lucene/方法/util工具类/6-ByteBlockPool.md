# 1 ByteBlockPool

数组链表结构。用于存储多组动态添加的Byte结构

首先它`PagedBytes` 类似，通过使用二维数组，存储数据。

在 `PagedBytes` 基础上，扩展一个类似链表的结构。用于存储多组的byte数据。



# 2 示例与原理

```java
    public static void main(String[] args)
    {

        ByteBlockPool byteBlockPool = new ByteBlockPool(new DirectAllocator());
        //分配一个空间为10的分配
        int index = byteBlockPool.newSlice(5);
        byte[] buffer = byteBlockPool.buffer;
        for (int i = index;index < 100000 ; i++)
        {
            //如果当前byte不为0 说明分配的空间已经用完，需要再分配新的空间
            if (buffer[i] != 0)
            {
                //分配新的空间，并返回，新空间的开始下标
                index = byteBlockPool.allocSlice(buffer, i);
                buffer = byteBlockPool.buffer;
                i = index-1;
                continue;
            }
            buffer[i] = 17;
        }
    }
```

上述代码流程为，通过newSlice 分配一个空间为5的slice。 

之后在每个分配的空间内存储值17。

直到分配的空间使用完，通过allocSlice 再重新分配新的空间。



内部byte数组变化如下：





![ByteBlocPool](ByteBlocPool.svg)



- 图一为 调用 newSlice 分配5个byte的空间。这时在数组的第5个元素处赋值16，标致分配的空间结束位置。
- 图二为 当在分配的空间使用完成时，数组 0到3 都存储的我们输入的值。
- 图三为 调用allocSlice 分配新空间，这时 原来空间的最后4个byte需要存储下一个空间开始的下标，所以数组1-4被修改为 5， 而原来2-4的3个值会被移到下一个空间，同时在新空间结束位置标志 17。
- 图四 与图二类似， 在新的空间内存储我们的输入内容。
- 图五、图六 依次类推，分配新空间与存储具体内容。



# 3 主要方法

## 3.1 主要成员变量

- buffers 二维数组，存储内容。
- buffer 当前正在使用的数组。
- *LEVEL_SIZE_ARRAY* = {5, 14, 20, 30, 40, 40, 80, 80, 120, 200} 新的分配分配的空间大小
- NEXT_LEVEL_ARRAY 每次分配新空间时使用的LEVEL_SIZE_ARRAY 数组下标



## 3.2 newSlice

```java
  public int newSlice(final int size) {
    if (byteUpto > BYTE_BLOCK_SIZE-size)
      nextBuffer();
    final int upto = byteUpto;
    byteUpto += size;
    buffer[byteUpto-1] = 16;
    return upto;
  }
```

nextBuffer 如果buffer 空间不足，分配新的buffer 。参考 `PagedBytes`逻辑

最后通过将新空间的最后一位`buffer[byteUpto-1] = 16` 赋值为16



## 3.3 allocSlice

```java
  public int allocSlice(final byte[] slice, final int upto) {
	//计算新的空间分配多少内容
    final int level = slice[upto] & 15;
    final int newLevel = NEXT_LEVEL_ARRAY[level];
    final int newSize = LEVEL_SIZE_ARRAY[newLevel];

    // buffer 不足，进行扩展
    if (byteUpto > BYTE_BLOCK_SIZE-newSize) {
      nextBuffer();
    }

    final int newUpto = byteUpto;
    final int offset = newUpto + byteOffset;
    byteUpto += newSize;

   
    // 将旧空间的最后3位赋值到新空间开始
    buffer[newUpto] = slice[upto-3];
    buffer[newUpto+1] = slice[upto-2];
    buffer[newUpto+2] = slice[upto-1];

    //进旧空间的最后4位赋值为新空间的开始下标
    slice[upto-3] = (byte) (offset >>> 24);
    slice[upto-2] = (byte) (offset >>> 16);
    slice[upto-1] = (byte) (offset >>> 8);
    slice[upto] = (byte) offset;
        
    // 在新空间的结尾写标志位
    buffer[byteUpto-1] = (byte) (16|newLevel);

    return newUpto+3;
  }

```

