# 1 DocValuesWriter

所有需要索引的字段，都是通过DocValuesWriter 先将索引字段值缓存到内存中，之后在当当前索引刷新时，调用 flush 方法，将数据落盘。



# 2 子类实现

对于不同类型的索引有着不同的实现类

-  NUMERIC 保存数值类型 org.apache.lucene.index.NumericDocValuesWriter
-  BINARY 保存字节数组，值可以大于32766字节  org.apache.lucene.index.BinaryDocValuesWriter
-  SORTED 保存字节数组，结果将an'z值必须小于等于32766字节 org.apache.lucene.index.SortedDocValuesWriter
-  SORTED_NUMERIC 多个值的数值类型，  org.apache.lucene.index.SortedNumericDocValuesWriter
-  SORTED_SET  多个值的去重的字节数组，值必须小于等于32766字节  org.apache.lucene.index.SortedSetDocValuesWriter





# 3 NumericDocValuesWriter

用于存储 数值类型的索引，主要用于缓存的成员变量有：

- pending = PackedLongValues.*deltaPackedBuilder*(PackedInts.*COMPACT*);-
- docsWithField = new DocsWithFieldSet();

分别为 `DocsWithFieldSet` 和 `PackedLongValues` 对象



## 3.1 addValue

为字段添加索引，

```java
  public void addValue(int docID, long value) {
    if (docID <= lastDocID) {
      throw new IllegalArgumentException("DocValuesField \"" + fieldInfo.name + "\" appears more than once in this document (only one value is allowed per field)");
    }

    pending.add(value);
    docsWithField.add(docID);

    updateBytesUsed();

    lastDocID = docID;
  }
```

我们看到添加值时，就是在`pending ` 和  `docsWithField ` 分别添加数据。





## 3.2 flush

```java
  @Override
  public void flush(SegmentWriteState state, Sorter.DocMap sortMap, DocValuesConsumer dvConsumer) throws IOException {
    if (finalValues == null) {
      finalValues = pending.build();
    }
    final NumericDVs sorted;
    if (sortMap != null) {
      NumericDocValues oldValues = new BufferedNumericDocValues(finalValues, docsWithField.iterator());
      sorted = sortDocValues(state.segmentInfo.maxDoc(), sortMap, oldValues);
    } else {
      sorted = null;
    }

    dvConsumer.addNumericField(fieldInfo,
                               new EmptyDocValuesProducer() {
                                 @Override
                                 public NumericDocValues getNumeric(FieldInfo fieldInfo) {
                                   if (fieldInfo != NumericDocValuesWriter.this.fieldInfo) {
                                     throw new IllegalArgumentException("wrong fieldInfo");
                                   }
                                   if (sorted == null) {
                                     return new BufferedNumericDocValues(finalValues, docsWithField.iterator());
                                   } else {
                                     return new SortingNumericDocValues(sorted);
                                   }
                                 }
                               });
  }
```



这里我们看到创建了 `EmptyDocValuesProducer` 用于数据遍历，而具体的写过程交给了`DocValuesConsumer dvConsumer`

## 3.3 BufferedNumericDocValues

通过 遍历 `pending ` 和  `docsWithField ` 对象，获取 文档id和具体的值



```java
    @Override
    public int docID() {
      return docsWithField.docID();
    }

    @Override
    public int nextDoc() throws IOException {
      int docID = docsWithField.nextDoc();
      if (docID != NO_MORE_DOCS) {
        value = iter.next();
      }
      return docID;
    }

    @Override
    public long longValue() {
      return value;
    }
```

这里主要关注3个方法， nextDoc。 获取下一个 文档和value 并返回 文档id。之后我们就可以通过 docID() 和 longValue()  获取我们想要的数据。



# 4 BinaryDocValuesWriter

## 4.1 数据存储

与NumericDocValuesWriter 类似， 文档id通过 `DocsWithFieldSet` 存储。但内容通过 `PagedBytes`进行存储.

同时通过`PackedLongValues` 记录每个byte数组的长度



## 4.2 BufferedBinaryDocValues

与NumericDocValuesWriter 区别在于获取数据

```java
    @Override
    public int nextDoc() throws IOException {
      int docID = docsWithField.nextDoc();
      if (docID != NO_MORE_DOCS) {
        int length = Math.toIntExact(lengthsIterator.next());
        value.setLength(length);
        bytesIterator.readBytes(value.bytes(), 0, length);
      }
      return docID;
    }
```

通过`PackedLongValues` 记录的数组长度，在`PagedBytes` 获取每个 bytes.



# 5 SortedDocValuesWriter



用于建立 BytesRef 类型的索引，在存储时，主要顺序是按照编码进行排序后的。

## 5.1 原理

对于BytesRef  使用`BytesRefHash ` 进行存储。 最后使用 `SortedDocValuesTermsEnum` 获取排序后的结果





# 6 SortedNumericDocValuesWriter

存储多值的 数值类型



## 6.1 主要成员变量

- ackedLongValues.Builder pending 存储所有的值
- PackedLongValues.Builder pendingCounts 存储没个id下值的个数
- DocsWithFieldSet docsWithField 存储文档id



## 6.2 主要方法



