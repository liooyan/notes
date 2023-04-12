# 1 Lucene80DocValuesConsumer

用于完成索引文件的落地，*写* dvd、dvm文件。



# 2  文件格式

dvd与dvm 文件存储的是列式的s

## 2.1 dvd

![dvd.drawio](dvd.drawio.svg)



- DOcsIDFileId  记录有当前字段的文档id
- FielDValues  字段的值，FielDValues   结构如下

![dvd-docId.drawio](dvd-docId.drawio.svg)







## 2.2 dvm





# 3 