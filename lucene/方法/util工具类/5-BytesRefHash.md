# 1 BytesRefHash

用于记录 BytesRef 数据的hash表。

通过**add **添加数据， 如果当前BytesRef第一次添加，返回一个自增的**id**。
如果当前BytesRef已经存在，返回的 是 