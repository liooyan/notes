# 1 ByteBlockPool

数组链表结构。用于存储多组动态添加的Byte结构

首先它`PagedBytes` 类似，通过使用二维数组，存储数据。

在 `PagedBytes` 基础上，扩展一个类似链表的结构。用于存储多组的byte数据。



# 2 示例

```java
    public static void main(String[] args)
    {

        ByteBlockPool byteBlockPool = new ByteBlockPool(new DirectAllocator());
        //分配一个空间为10的分配
        int index = byteBlockPool.newSlice(10);
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
            buffer[i] = 0x11;
        }
    }
```

上图示示例li





BlockPool将数组分为一个一个slice数据片,假如有数据存入时会分配一个slice，如下图1。

当有新的一组数据添加时，会分配一个新的slice 用于存储新分组的数据，如下图2.

当第一个slice数据存储慢后，会分配一个更大的slice空间，同时上一个空间的最后4位字节将指向新的空间下标。这样就形成了一个链表结构，如图3。

以此类推，如图4、图5。



![ByteBlocPool](ByteBlocPool.svg)



# 2 主要方法