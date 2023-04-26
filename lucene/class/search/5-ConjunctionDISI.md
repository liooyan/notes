# 1 ConjunctionDISI

当 BooleanQuery 的 `MUST` 条件使用该方法实现。此类将多个must 条件共同迭代。



# 2 构造函数



```java
  private ConjunctionDISI(List<? extends DocIdSetIterator> iterators) {
    assert iterators.size() >= 2;

    // Sort the array the first time to allow the least frequent DocsEnum to
    // lead the matching.
     // 将数量少的字段放在前面，减少遍历计算
    CollectionUtil.timSort(iterators, new Comparator<DocIdSetIterator>() {
      @Override
      public int compare(DocIdSetIterator o1, DocIdSetIterator o2) {
        return Long.compare(o1.cost(), o2.cost());
      }
    });
    lead1 = iterators.get(0);
    lead2 = iterators.get(1);
    others = iterators.subList(2, iterators.size()).toArray(new DocIdSetIterator[0]);
  }
```

- 参数为多个DocIdSetIterator ，用于返回文档id 迭代器。



# 3 主要方法

## 3.1 doNext(int doc)

获取下一个多个迭代器共有的文档id

```java
  private int doNext(int doc) throws IOException {
    advanceHead: for(;;) {
      assert doc == lead1.docID();

      // find agreement between the two iterators with the lower costs
      // we special case them because they do not need the
      // 'other.docID() < doc' check that the 'others' iterators need
      //获取文档2直到 doc id的值
      final int next2 = lead2.advance(doc);
      //如果不相等，就继续重新获取第一个文档的值，直到文档1和文档2 的值相同
      if (next2 != doc) {
        doc = lead1.advance(next2);
        if (next2 != doc) {
          continue;
        }
      }

      // then find agreement with other iterators
      //同样逻辑，对比其他字段的文档id
      for (DocIdSetIterator other : others) {
        // other.doc may already be equal to doc if we "continued advanceHead"
        // on the previous iteration and the advance on the lead scorer exactly matched.
        if (other.docID() < doc) {
          final int next = other.advance(doc);

          if (next > doc) {
            // iterator beyond the current doc - advance lead and continue to the new highest doc.
            doc = lead1.advance(next);
            continue advanceHead;
          }
        }
      }

      // success - all iterators are on the same doc
      return doc;
    }
  }

```





# 4 子类 



## 4.1 ConjunctionTwoPhaseIterator

继承自`TwoPhaseIterator`，添加过滤条件

```java
    @Override
    public boolean matches() throws IOException {
      for (TwoPhaseIterator twoPhaseIterator : twoPhaseIterators) { // match cheapest first
        if (twoPhaseIterator.matches() == false) {
          return false;
        }
      }
      return true;
    }
```

