# 1 NUMERIC 格式

本文档在 Lucene80DocValuesConsumer 的文档基础上，描述关于NUMERIC 字段类型的`FieldValues`  编码。



# 2 FieldValues

这里FieldValues 存储的就是一个数值数组，但为了尽可能的提高压缩率，需要确保保存的数据尽可能的小，这样才可以通过类似 `PackedInts` 的方式减少存储空间。所以采取以下方式：

- 所有数据存储的都是与最小值的差值
- 除了存储差







# 3 dvm- FieldValues  部分

dvm文档除了 DocsIDFileId  部分外，还有描述FieldValues相关字段的索引，具体如下：

