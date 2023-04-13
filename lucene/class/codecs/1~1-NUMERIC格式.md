# 1 NUMERIC 格式

本文档在 Lucene80DocValuesConsumer 的文档基础上，描述关于NUMERIC 字段类型的`FieldValues`  编码。



# 2 FieldValues

这里FieldValues 存储的就是一个数值数组，但为了尽可能的提高压缩率，需要确保保存的数据尽可能的小，这样才可以通过类似 `PackedInts` 的方式减少存储空间。所以采取以下方式：

- 所有数据存储的都是与最小值的差值
- 除了存储差值，还除以这些数值的最大公约数，以减小其数值
- 为可使最大公约数尽可能的大，可能采取分块的方式，这样在局部范围内。才有可能使最大公约数值越大。当然如果分块将减少的比例在0.9以上，则不会被分块。



以上就是对于NUMERIC 格式 的FieldValues。其数据最后为压缩后的数值。我们主要关注点在dvm上，观察dvm。记录哪些信息，方便我们解析FieldValues







# 3 dvm- FieldValues  部分

dvm文档除了 DocsIDFileId  部分外，还有描述FieldValues相关字段的索引，具体如下：

![dvm-number](dvm-number.svg)



- blockCount ： FieldValues 分块数量，如果不分块。则为-1
- numBitsPerValue ： 数组的每个元素，使用几个bit位来存储
- 
