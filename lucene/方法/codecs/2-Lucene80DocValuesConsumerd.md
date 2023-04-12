# 1 Lucene80DocValuesConsumer

用于完成索引文件的落地，*写* dvd、dvm文件。



# 2  文件格式

dvd与dvm 文件存储的是列式的索引信息。

在dvd文件中，按照不同字段分为如下结构：

![dvd.drawio](dvd.drawio.svg)

每个字段就是一个docsIDField 和FielValues的组合，当前段有几个字段就有几种这样的组合。

- DocsIDFileId  记录有当前字段的文档id
- FielDValues  字段的值，FielDValues   结构如下



## 2.1 



## 2.1 dvd





![dvd-docId.drawio](dvd-docId.drawio.svg)







## 2.2 dvm





# 3 