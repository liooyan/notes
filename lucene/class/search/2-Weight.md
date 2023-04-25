# 1 Weight

在Query 中通过 createWeight 生成此对象，在此对象中获取查询结果。包括：

- scorer  返回当前段文件符合当前匹配结果的文档id迭代器
- matches 判断当前段的指定文档id数据是否符合当前查询条件





其中 scorer   为抽象方法，子类实现





# 2 Scorer

scorer 方法返回的对象，主要包括以下方法定义：

- DocIdSetIterator iterator() 获取符合当前条件的文档id 迭代器
- TwoPhaseIterator twoPhaseIterator() 返回  `TwoPhaseIterator `对象



## 2.1 ConstantScoreScorer

Scorer 的具体实现

```java
  public ConstantScoreScorer(Weight weight, float score, ScoreMode scoreMode, TwoPhaseIterator twoPhaseIterator) {
    super(weight);
    this.score = score;
    this.scoreMode = scoreMode;
    if (scoreMode == ScoreMode.TOP_SCORES) {
      this.approximation = new DocIdSetIteratorWrapper(twoPhaseIterator.approximation());
      this.twoPhaseIterator = new TwoPhaseIterator(this.approximation) {
        @Override
        public boolean matches() throws IOException {
          return twoPhaseIterator.matches();
        }

        @Override
        public float matchCost() {
          return twoPhaseIterator.matchCost();
        }
      };
    } else {
      this.approximation = twoPhaseIterator.approximation();
      this.twoPhaseIterator = twoPhaseIterator;
    }
    this.disi = TwoPhaseIterator.asDocIdSetIterator(this.twoPhaseIterator);
  }
```

构造函数中 TwoPhaseIterator 是通过参数传入的，而 iterator() 返回的是disi，是通过 TwoPhaseIterator 获取的







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



# 4 TwoPhaseIterator

提供 matches 与 matchCost 方法，由子类实现，具体不同的过滤条件，有不同的实现，

但构造函数需要一个DocIdSetIterator ， 这样就可以与其他Iterator 共享同一个迭代其，方便多条件查询





## 4.1 TwoPhaseIterator.asDocIdSetIterator

在 ConstantScoreScorer 中我们知道它的迭代器就是通过这个方法获取的。

```java
  public static DocIdSetIterator asDocIdSetIterator(TwoPhaseIterator twoPhaseIterator) {
    return new TwoPhaseIteratorAsDocIdSetIterator(twoPhaseIterator);
  }
```

也就是TwoPhaseIterator的子类TwoPhaseIteratorAsDocIdSetIterator，其主要方法为：

```java
    private int doNext(int doc) throws IOException {
      for (;; doc = approximation.nextDoc()) {
        if (doc == NO_MORE_DOCS) {
          return NO_MORE_DOCS;
        } else if (twoPhaseIterator.matches()) {
          return doc;
        }
      }
    }
```

这里我们看到迭代器的next 方法最后的matches 也使用的是 twoPhaseIterator 中的方法
