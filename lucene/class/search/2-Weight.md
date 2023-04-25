# 1 Weight

在Query 中通过 createWeight 生成此对象，在此对象中获取查询结果。包括：

- scorer  返回当前段文件符合当前匹配结果的文档id迭代器
- matches 判断当前段的指定文档id数据是否符合当前查询条件





其中 scorer   为抽象方法，子类实现





# 2 Scorer

scorer 方法返回的对象，主要包括以下方法定义：

- DocIdSetIterator iterator() 获取符合当前条件的文档id 迭代器
- TwoPhaseIterator twoPhaseIterator() 返回  `TwoPhaseIterator `对象





# 3 matches

```java
  public Matches matches(LeafReaderContext context, int doc) throws IOException {
    ScorerSupplier scorerSupplier = scorerSupplier(context);
    if (scorerSupplier == null) {
      return null;
    }
    Scorer scorer = scorerSupplier.get(1);
    final TwoPhaseIterator twoPhase = scorer.twoPhaseIterator();
    if (twoPhase == null) {
      if (scorer.iterator().advance(doc) != doc) {
        return null;
      }
    }
    else {
      if (twoPhase.approximation().advance(doc) != doc || twoPhase.matches() == false) {
        return null;
      }
    }
    return MatchesUtils.MATCH_WITH_NO_TERMS;
  }
```

这里同样调用  scorer  方法。分为两者情况

- 包含TwoPhaseIterator  对象，则使用 TwoPhaseIterator  对象的matches 方法
- 不包含 TwoPhaseIterator 对象，则将获取到的迭代器 推进到我们需要的文档id。判断是否有当前id



