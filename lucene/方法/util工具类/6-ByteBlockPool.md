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
- 图三为 调用allocSlice 分配新空间，这时 原来空间的最后4个byte需要存储下一个空间开始的下标，所以数组1-4被修改为00



# 2 主要方法