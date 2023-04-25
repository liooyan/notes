# 1 SortedNumericDocValuesRangeQuery

 SortedNumeric 类型数据的 range 过滤条件。



# 2 构造函数

SortedNumericDocValuesRangeQuery 是一个抽象类，接收3个参数

```java
  SortedNumericDocValuesRangeQuery(String field, long lowerValue, long upperValue) {
    this.field = Objects.requireNonNull(field);
    this.lowerValue = lowerValue;
    this.upperValue = upperValue;
  }
```

- field 字段名称
- lowerValue 最大值
- upperValue 最小值

其中 获取字段值的方法为抽象方法未实现

```java
  abstract SortedNumericDocValues getValues(LeafReader reader, String field) throws IOException;
```



# 3 主要方法

## 3.1 createWeight

这里我们主要关注 scorer 方法的实现，也就是我们在`Weight`类中分析的核心方法

```java
      @Override
      public Scorer scorer(LeafReaderContext context) throws IOException {
        //通过getValues 获取加载数据的迭代器
        SortedNumericDocValues values = getValues(context.reader(), field);
        if (values == null) {
          return null;
        }
        final NumericDocValues singleton = DocValues.unwrapSingleton(values);
        final TwoPhaseIterator iterator;
        // 实现 TwoPhaseIterator的 matches 方法。就是通过判断 value 和lowerValue，upperValue 的大小
        if (singleton != null) {
          iterator = new TwoPhaseIterator(singleton) {
            @Override
            public boolean matches() throws IOException {
              final long value = singleton.longValue();
              return value >= lowerValue && value <= upperValue;
            }

            @Override
            public float matchCost() {
              return 2; // 2 comparisons
            }
          };
        } else {
          iterator = new TwoPhaseIterator(values) {
            @Override
            public boolean matches() throws IOException {
              for (int i = 0, count = values.docValueCount(); i < count; ++i) {
                final long value = values.nextValue();
                if (value < lowerValue) {
                  continue;
                }
                // Values are sorted, so the first value that is >= lowerValue is our best candidate
                return value <= upperValue;
              }
              return false; // all values were < lowerValue
            }

            @Override
            public float matchCost() {
              return 2; // 2 comparisons
            }
          };
        }
        return new ConstantScoreScorer(this, score(), scoreMode, iterator);
      }
```





# 4 创建

在 NumericDocValuesField 中， 创建了SortedNumericDocValuesRangeQuery 子类，实现了 getValues 数据加载。

```java
  public static Query newSlowRangeQuery(String field, long lowerValue, long upperValue) {
    return new SortedNumericDocValuesRangeQuery(field, lowerValue, upperValue) {
      @Override
      SortedNumericDocValues getValues(LeafReader reader, String field) throws IOException {
        NumericDocValues values = reader.getNumericDocValues(field);
        if (values == null) {
          return null;
        }
        return DocValues.singleton(values);
      }
    };
  }
```

这里我们看到使用的就是 reader.getNumericDocValues 。也就是 `Lucene80DocValuesProducer` 类中加载数据的方法。
