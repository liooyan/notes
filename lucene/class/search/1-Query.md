# 1 Query

查询条件的基础类，其实现有

- TermQuery
- BooleanQuery
- WildcardQuery
- PhraseQuery
- PrefixQuery
- MultiPhraseQuery
- FuzzyQuery
- RegexpQuery
- TermRangeQuery
- PointRangeQuery
- ConstantScoreQuery
- DisjunctionMaxQuery
- MatchAllDocsQuery



# 2 主要方法

通过 `public Weight createWeight(IndexSearcher searcher, ScoreMode scoreMode, float boost)` 获取 `Weight` 对象

测试用例如下：

```java
 IndexReader reader = DirectoryReader.open(directory);
        IndexSearcher searcher = new IndexSearcher(reader);
        List<LeafReaderContext> leaves = reader.getContext().leaves();
        Query q = NumericDocValuesField.newSlowRangeQuery("visit", 32, 66);

        Weight weight = q.createWeight(searcher, ScoreMode.COMPLETE_NO_SCORES, 1);
        //        weight.scorer(reader.getContext());
        //        weight.
        long startTime = System.currentTimeMillis();
        for (LeafReaderContext leaf : leaves)
        {
            Scorer scorer = weight.scorer(leaf);
            if (scorer == null)
            {
                continue;
            }
            DocIdSetIterator iterator = scorer.iterator();
            int doc;
            while ((doc = iterator.nextDoc()) != DocIdSetIterator.NO_MORE_DOCS)
            {
                System.out.println(doc);
            }
        }
```

