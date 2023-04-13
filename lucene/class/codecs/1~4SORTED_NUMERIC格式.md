# 1 ORTED_NUMERIC格式

本文档在 Lucene80DocValuesConsumer 的文档基础上，描述关于ORTED_NUMERIC字段类型的`FieldValues`  编码。





#  FieldValues

ORTED_NUMERIC文档为存储多个数值属性索引，所以它有NUMERIC 格式的所有内容，同时在NUMERIC 格式基础上添加关于每个文档对应的数量。

添加一个addresses 数组，addresses每个元素为当前文档id的值，在values数组的下标







# 3 dvm- FieldValues  部分

在NUMERIC  格式基础上添加关于address数组的描述

- start ：address数组开始位置
- BLOCK_SHIFT
- pointer： address数组占用大小