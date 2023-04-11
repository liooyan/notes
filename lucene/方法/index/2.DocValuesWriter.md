# 1 DocValuesWriter

所有需要索引的字段，都是通过DocValuesWriter 先将索引字段值缓存到内存中，之后在当当前索引刷新时，调用 flush 方法，将数据落盘。



# 2 子类实现

对于不同类型的索引有着不同的实现类

-  NUMERIC 保存数值类型 org.apache.lucene.index.NumericDocValuesWriter
-  BINARY 保存字节数组，值可以大于32766字节  org.apache.lucene.index.BinaryDocValuesWriter
-  SORTED 多值的字节数组，值必须小于等于32766字节 org.apache.lucene.index.SortedDocValuesWriter
-  SORTED_NUMERIC 多个值的数值类型，  org.apache.lucene.index.SortedNumericDocValuesWriter
-  SORTED_SET  多个值的去重的字节数组，值必须小于等于32766字节  org.apache.lucene.index.SortedSetDocValuesWriter





# 3 NumericDocValuesWriter
