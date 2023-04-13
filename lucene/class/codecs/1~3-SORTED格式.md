# 1 SORTED格式

本文档在 Lucene80DocValuesConsumer 的文档基础上，描述关于SORTED字段类型的`FieldValues`  编码。

# 2 FieldValues

SORTED 为短语格式的索引，ord





# 3 dvm- FieldValues  部分



![dvm-sorted](dvm-sorted.svg)



- numberOfBitsPerOrd ： 一个ord占用的bit位
- ordStartOffset： ord数组在dvd文件开始位置
- ordPointer： ord数组占用dvd大小
- code： 压缩格式
- blockShift： 分片方式
- maxLength：values中占用最大的长度
- valueStartOffset： value数组在dvd开始位置
- valuePointer： value占用dvd大小
- addressStartOffset：address数组在dvd 文件开始位置
- addressPointer ： address数组在dvd占用大小