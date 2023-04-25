```java
    IndexReader reader = DirectoryReader.open(directory);
    IndexSearcher searcher = new IndexSearcher(reader);
    List<LeafReaderContext> leaves = reader.getContext().leaves();

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
    System.out.println("查询时间：" + (System.currentTimeMillis() - startTime));
```