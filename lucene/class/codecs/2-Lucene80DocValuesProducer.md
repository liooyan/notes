# 1 Lucene80DocValuesProducer

加载 索引数据，读取 dvd与 dvm





# 2 构造函数

```java
  Lucene80DocValuesProducer(SegmentReadState state, String dataCodec, String dataExtension, String metaCodec, String metaExtension) throws IOException {
    String metaName = IndexFileNames.segmentFileName(state.segmentInfo.name, state.segmentSuffix, metaExtension);
    this.maxDoc = state.segmentInfo.maxDoc();
    ramBytesUsed = RamUsageEstimator.shallowSizeOfInstance(getClass());

    // read in the entries from the metadata file.
    try (ChecksumIndexInput in = state.directory.openChecksumInput(metaName, state.context)) {
      Throwable priorE = null;
      
      try {
        version =
            CodecUtil.checkIndexHeader(
                in,
                metaCodec,
                Lucene80DocValuesFormat.VERSION_START,
                Lucene80DocValuesFormat.VERSION_CURRENT,
                state.segmentInfo.getId(),
                state.segmentSuffix);

        readFields(state.segmentInfo.name, in, state.fieldInfos);
      } catch (Throwable exception) {
        priorE = exception;
      } finally {
        CodecUtil.checkFooter(in, priorE);
      }
    }
```

在构造函数中加载dvm文件，加载meta信息，调用 readFields 解析字段



# 3 readFields

```java
  private void readFields(String segmentName, ChecksumIndexInput meta, FieldInfos infos) throws IOException {
    for (int fieldNumber = meta.readInt(); fieldNumber != -1; fieldNumber = meta.readInt()) {
      FieldInfo info = infos.fieldInfo(fieldNumber);
      if (info == null) {
        throw new CorruptIndexException("Invalid field number: " + fieldNumber, meta);
      }
      byte type = meta.readByte();
      if (type == Lucene80DocValuesFormat.NUMERIC) {
        numerics.put(info.name, readNumeric(meta));
      } else if (type == Lucene80DocValuesFormat.BINARY) {
        final boolean compressed;
        if (version >= Lucene80DocValuesFormat.VERSION_CONFIGURABLE_COMPRESSION) {
          String value = info.getAttribute(Lucene80DocValuesFormat.MODE_KEY);
          if (value == null) {
            throw new IllegalStateException(
                "missing value for "
                    + Lucene80DocValuesFormat.MODE_KEY
                    + " for field: "
                    + info.name
                    + " in segment: "
                    + segmentName);
          }
          Lucene80DocValuesFormat.Mode mode = Lucene80DocValuesFormat.Mode.valueOf(value);
          compressed = mode == Lucene80DocValuesFormat.Mode.BEST_COMPRESSION;
        } else {
          compressed = version >= Lucene80DocValuesFormat.VERSION_BIN_COMPRESSED;
        }
        binaries.put(info.name, readBinary(meta, compressed));
      } else if (type == Lucene80DocValuesFormat.SORTED) {
        sorted.put(info.name, readSorted(meta));
      } else if (type == Lucene80DocValuesFormat.SORTED_SET) {
        sortedSets.put(info.name, readSortedSet(meta));
      } else if (type == Lucene80DocValuesFormat.SORTED_NUMERIC) {
        sortedNumerics.put(info.name, readSortedNumeric(meta));
      } else {
        throw new CorruptIndexException("invalid type: " + type, meta);
      }
    }
  }
```

不同的字段类型，通过不同的方法 将meta 内容写入到 不同map中

- numerics   
- binaries
- sorted
- sortedSets
- sortedNumerics



# 4 获取 DocValuesIterator

在加载完 meta 信息后，可调用各自的数据加载方法，获取 DocValuesIterator 获取字段值的迭代器



## 4.1 getNumeric

获取 numerics    类型的数据，其返回的是 `NumericDocValues`

根据不同的存储方法，加载方式也不同，逻辑为：



通过IndexedDISI 获取id 的加载

```java
final IndexedDISI disi = new IndexedDISI(data, entry.docsWithFieldOffset, entry.docsWithFieldLength,
          entry.jumpTableEntryCount, entry.denseRankPower, entry.numValues);
```





通过 `LongValues` 加载字段值，它是实时得从文本中加载数据。

```java
final LongValues values = DirectReader.getInstance(slice, entry.bitsPerValue);
```



NumericDocValues 主要方法如下：

```java

//获取下一个id
    @Override
    public int nextDoc() throws IOException {
      return disi.nextDoc();
    }

//获取下一个值
    @Override
    public long longValue() throws IOException {
        return mul * values.get(disi.index()) + delta;
     }

```





## 4.2 getSortedNumeric

基本与 getNumeric 相同，只是多了一个address 数组

```java
    final RandomAccessInput addressesInput = data.randomAccessSlice(entry.addressesOffset, entry.addressesLength);
```





同样 SortedNumericDocValues 的方法也将不同

```java
        @Override
        public boolean advanceExact(int target) throws IOException {
          start = addresses.get(target);
          end = addresses.get(target + 1L);
          count = (int) (end - start);
          doc = target;
          return true;
        }

        @Override
        public long nextValue() throws IOException {
          return values.get(start++);
        }
```



