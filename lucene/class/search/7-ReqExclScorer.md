# 1 ReqExclScorer

当 BooleanQuery 的 `MUST_NOT` 条件使用该方法实现。需要两个参数

- reqScorer 需要满足的条件
- exclScorer 需要排查的条件





# 2 初始化

```java
  public ReqExclScorer(Scorer reqScorer, Scorer exclScorer) {
    super(reqScorer.weight);
    this.reqScorer = reqScorer;
    reqTwoPhaseIterator = reqScorer.twoPhaseIterator();
    if (reqTwoPhaseIterator == null) {
      reqApproximation = reqScorer.iterator();
    } else {
      reqApproximation = reqTwoPhaseIterator.approximation();
    }
    exclTwoPhaseIterator = exclScorer.twoPhaseIterator();
    if (exclTwoPhaseIterator == null) {
      exclApproximation = exclScorer.iterator();
    } else {
      exclApproximation = exclTwoPhaseIterator.approximation();
    }
  }

```





# 3 matches

```java
        @Override
        public boolean matches() throws IOException {
           //获取当前must的id
          final int doc = reqApproximation.docID();
          // check if the doc is not excluded
            // 获取当前排查的id
          int exclDoc = exclApproximation.docID();
            //如果当前排查id小于mustid，迭代至当前must的下一个id
          if (exclDoc < doc) {
            exclDoc = exclApproximation.advance(doc);
          }
            // 不相等，就排查
          if (exclDoc != doc) {
            return matchesOrNull(reqTwoPhaseIterator);
          }
          return matchesOrNull(reqTwoPhaseIterator) && !matchesOrNull(exclTwoPhaseIterator);
        }
```

