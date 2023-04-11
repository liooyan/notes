# 1 ByteBlockPool

数组链表结构。用于存储多组动态添加的Byte结构

首先它`PagedBytes` 类似，通过使用二维数组，存储数据。

在 `PagedBytes` 基础上，扩展一个类似链表的结构。用于存储多组的byte数据。



ByteBlockPool将数组分为一个一个slice数据片,假如有数据存入时会分配一个slice，如下



![ByteBlocPool](ByteBlocPool.svg)