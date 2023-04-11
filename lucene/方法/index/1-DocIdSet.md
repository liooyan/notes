# 1 DocIdSet

用于存储文档id 的数据集，抽象类，主要方法：

-  DocIdSetIterator iterator() 获取文档id 的迭代器

主要子类 DocsWithFieldSet



# 2  实现原理

文档id分为两者情况记录。

- 文档id从0开始，都是连续的，没有间断。这种情况无需额外空间，只需要记录最大的文档id即可
- 文档id中间有间断，则使用FixedBitSet进行记录



# 3 主要方法

## 3.1 add 添加一个新的id

```java

  void add(int docID) {
    if (docID <= lastDocId) {
      throw new IllegalArgumentException("Out of order doc ids: last=" + lastDocId + ", next=" + docID);
    }
    if (set != null) {
      set = FixedBitSet.ensureCapacity(set, docID);
      set.set(docID);
     // 传入的文档id与上一个存储的文档id不连续，则使用fixedBitSet 存储
    } else if (docID != cost) {
      // migrate to a sparse encoding using a bit set
      set = new FixedBitSet(docID + 1);
      set.set(0, cost);
      set.set(docID);
    }
    lastDocId = docID;
    cost++;
  }
```

这里我们看到 如果文档id一直连续，则只记录 lastDocId，否则就需要使用`FixedBitSet`存储数据



## 3.2 iterator 获取文档id的迭代器

```java
  public DocIdSetIterator iterator() {
    return set != null ? new BitSetIterator(set, cost) : DocIdSetIterator.all(cost);
  }
```

这里同样分为 连续与非连续两者情况





### 3.2.1 连续 DocIdSetIterator.*all*

```java
  public static final DocIdSetIterator all(int maxDoc) {
    return new DocIdSetIterator() {
      int doc = -1;

      @Override
      public int docID() {
        return doc;
      }

      @Override
      public int nextDoc() throws IOException {
        return advance(doc + 1);
      }

      @Override
      public int advance(int target) throws IOException {
        doc = target;
        if (doc >= maxDoc) {
          doc = NO_MORE_DOCS;
        }
        return doc;
      }

      @Override
      public long cost() {
        return maxDoc;
      }
    };
  }
```

对于连续的id，就是从0到lastDocId ，依次递增返回我们需要的文档id



### 3.2.2 非连续 BitSetIterator

```java
  public int nextDoc() {
    return advance(doc + 1);
  }

  public int advance(int target) {
    if (target >= length) {
      return doc = NO_MORE_DOCS;
    }
    return doc = bits.nextSetBit(target);
  }
```

这里调用的是 FixedBitSet的nextSetBit 方法