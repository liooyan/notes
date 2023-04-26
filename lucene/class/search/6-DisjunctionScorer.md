# 1 DisjunctionScorer

当 BooleanQuery 的 `SHOULD` 条件使用该方法实现。此类的子类 `TwoPhase` 将多个SHOULD 条件共同迭代。



# 2 DisjunctionDISIApproximation

获取联合字段字段id的迭代器

```java
  @Override
  public int nextDoc() throws IOException {
    DisiWrapper top = subIterators.top();
    final int doc = top.doc;
    do {
      top.doc = top.approximation.nextDoc();
      top = subIterators.updateTop();
    } while (top.doc == doc);

    return top.doc;
  }
```

多个字段同时迭代，通过 subIterators.updateTop 却换不同的字段





# 3 TwoPhase

过滤条件判断

```java
    public boolean matches() throws IOException {
      verifiedMatches = null;
      unverifiedMatches.clear();
      //获取当前有当前id的所有字段迭代器
      for (DisiWrapper w = subScorers.topList(); w != null; ) {
        DisiWrapper next = w.next;
        
        if (w.twoPhaseView == null) {
          // implicitly verified, move it to verifiedMatches
          w.next = verifiedMatches;
          verifiedMatches = w;
          
          if (needsScores == false) {
            // we can stop here
            return true;
          }
        } else {
          unverifiedMatches.add(w);
        }
        w = next;
      }
      
      if (verifiedMatches != null) {
        return true;
      }
      
      // verify subs that have an two-phase iterator
      // least-costly ones first
      // 进行判断，并返回结果
      while (unverifiedMatches.size() > 0) {
        DisiWrapper w = unverifiedMatches.pop();
        if (w.twoPhaseView.matches()) {
          w.next = null;
          verifiedMatches = w;
          return true;
        }
      }
      
      return false;
    }
```

