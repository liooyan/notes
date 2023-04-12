# 1 Lucene80DocValuesConsumer

用于完成索引文件的落地，*写* dvd、dvm文件。



# 2  文件格式

dvd与dvm 文件存储的是列式的索引信息。

在dvd文件中，按照不同字段分为如下结构：

![dvd.drawio](dvd.drawio.svg)

每个字段就是一个docsIDField 和FielValues的组合，当前段有几个字段就有几种这样的组合。

- DocsIDFileId  记录有当前字段的文档id
- FielDValues  字段的值，FielDValues   结构如下



## 2.1 DocsIDFileId  

DocsIDFileId   存储的是哪些文档有当前字段，记录的是文档id值。

但又分为 3种不同情况

- 所有文档都有当前字段
- 所有文档都没有当前字段
- 部分文档





## 2.1 dvd





![dvd-docId.drawio](dvd-docId.drawio.svg)







## 2.2 dvm





# 3 