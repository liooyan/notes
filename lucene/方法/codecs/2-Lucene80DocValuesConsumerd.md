# 1 Lucene80DocValuesConsumer

用于完成索引文件的落地，*写* dvd、dvm文件。



# 2  文件格式

dvd与dvm 文件存储的是列式的索引信息，在dvd文件中，按照不同zi'd

## 2.1 dvd

![dvd.drawio](dvd.drawio.svg)



- DocsIDFileId  记录有当前字段的文档id
- FielDValues  字段的值，FielDValues   结构如下

![dvd-docId.drawio](dvd-docId.drawio.svg)







## 2.2 dvm





# 3 