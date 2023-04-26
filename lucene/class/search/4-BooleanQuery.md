# 1 BooleanQuery

Bool 过滤。







# 2 Boolean2ScorerSupplier

用于创建 Boolean 条件的 Scorer 对象

而 Scorer 对象则包含了具体的过滤迭代器，比如

- MUST  ---> ConjunctionDISI 对象
- SHOULD   --->  DisjunctionScorer 对象
- MUST_NOT ---> ReqExclScorer 对象





# 3 getInternal(long leadCost)

获取 Boolean条件的 Scorer 对象。参数为最大遍历的文档id数量

```java
  private Scorer getInternal(long leadCost) throws IOException {
    // three cases: conjunction, disjunction, or mix
    leadCost = Math.min(leadCost, cost());

    // pure conjunction
    if (subs.get(Occur.SHOULD).isEmpty()) {
      return excl(req(subs.get(Occur.FILTER), subs.get(Occur.MUST), leadCost), subs.get(Occur.MUST_NOT), leadCost);
    }

    // pure disjunction
    if (subs.get(Occur.FILTER).isEmpty() && subs.get(Occur.MUST).isEmpty()) {
      return excl(opt(subs.get(Occur.SHOULD), minShouldMatch, scoreMode, leadCost), subs.get(Occur.MUST_NOT), leadCost);
    }

    // conjunction-disjunction mix:
    // we create the required and optional pieces, and then
    // combine the two: if minNrShouldMatch > 0, then it's a conjunction: because the
    // optional side must match. otherwise it's required + optional

    if (minShouldMatch > 0) {
      Scorer req = excl(req(subs.get(Occur.FILTER), subs.get(Occur.MUST), leadCost), subs.get(Occur.MUST_NOT), leadCost);
      Scorer opt = opt(subs.get(Occur.SHOULD), minShouldMatch, scoreMode, leadCost);
      return new ConjunctionScorer(weight, Arrays.asList(req, opt), Arrays.asList(req, opt));
    } else {
      assert scoreMode.needsScores();
      return new ReqOptSumScorer(
          excl(req(subs.get(Occur.FILTER), subs.get(Occur.MUST), leadCost), subs.get(Occur.MUST_NOT), leadCost),
          opt(subs.get(Occur.SHOULD), minShouldMatch, scoreMode, leadCost), scoreMode);
    }
  }
```



