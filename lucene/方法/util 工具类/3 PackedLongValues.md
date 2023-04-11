# 1 PackedLongValues

用于存储一批 long 数组，并将其压缩。

其中压缩使用的是PackedInts 类，进行压缩。





# 2 实现原理



在此类中主要有两个内部类

- Builder  用于创建PackedLongValues对象，向其添加数据
- Iterator 用于迭代当前PackedLongValues 存储的数据

测试用例：

```java
    static Random random = new Random();
    public static void main(String[] args) {
        Builder builder =  PackedLongValues.packedBuilder(256, 0.0F);


        for (int i = 0; i < 1000000 ; i++) {
            builder.add(random.nextInt(10000));
        }
        PackedLongValues packedLongValues = builder.build();

        PackedLongValues.Iterator iterator = packedLongValues.iterator();

        while (iterator.hasNext()){
            System.out.println(iterator.next());
        }

    }
```



Builder 对于数据的存储主要是按照 `块`进行存储。 通过构造函数指定一块的大小，

如果当前存储的数据小于块的大小，则零时存储到`long[] pending`数组中

如果当前存储数据积累到块的大小，则将`long[] pending` 的数据打包到 `PackedInts.Reader`.

并将其保存到`PackedInts.Reader[] values`对象中。同时将pending 数据清零，用于下一批数据存储

而PackedInts.Reader 就是通过`PackedInts `类创建



# 3 主要方法

## 3.1 builder.add  添加数据

```java
    public Builder add(long l) {
      if (pending == null) {
        throw new IllegalStateException("Cannot be reused after build()");
      }
       //如果当前存储数据已经在pending 存储满就需要 打包压缩数据
      if (pendingOff == pending.length) {
        // 如果 PackedInts.Reader[] values已经使用完，就需要扩容
        if (values.length == valuesOff) {
          final int newLength = ArrayUtil.oversize(valuesOff + 1, 8);
          grow(newLength);
        }
          // 压缩数据
        pack();
      }
      pending[pendingOff++] = l;
      size += 1;
      return this;
    }

```



对于数据的添加就是`pending[pendingOff++] = l` 将数据存储到pending，而主要我们关注的是 `pack` 数据压缩

```java
    void pack(long[] values, int numValues, int block, float acceptableOverheadRatio) {
      assert numValues > 0;
      // compute max delta
      long minValue = values[0];
      long maxValue = values[0];
      for (int i = 1; i < numValues; ++i) {
        minValue = Math.min(minValue, values[i]);
        maxValue = Math.max(maxValue, values[i]);
      }

      // build a new packed reader
      if (minValue == 0 && maxValue == 0) {
        this.values[block] = new PackedInts.NullReader(numValues);
      } else {
        final int bitsRequired = minValue < 0 ? 64 : PackedInts.bitsRequired(maxValue);
        final PackedInts.Mutable mutable = PackedInts.getMutable(numValues, bitsRequired, acceptableOverheadRatio);
        for (int i = 0; i < numValues; ) {
          i += mutable.set(i, values, i, numValues - i);
        }
        this.values[block] = mutable;
      }
    }
```

这里我们看到创建了  PackedInts.Mutable 对象进行了数据压缩





## 3.2 Iterator 

对于数据的遍历，主要关注几个成员变量

- vOff 已经遍历过的数据
- currentCount 当前对象，存储数的总数量
- currentValues 当前正在变量的数组



## 3.2.1 Iterator.next

```java
    public final long next() {
      assert hasNext();
      long result = currentValues[pOff++];
      if (pOff == currentCount) {
        vOff += 1;
        pOff = 0;
        fillBlock();
      }
      return result;
    }

```

这里我们看到就是通过vOff 在 currentCount  ，当 currentCount   使用完后，就需要fillBlock 获取下一个PackedInts.Mutable  对象的数据，替换掉currentValues



## 3.2.2 fillBlock

```java
  int decodeBlock(int block, long[] dest) {
    final PackedInts.Reader vals = values[block];
    final int size = vals.size();
    for (int k = 0; k < size; ) {
      k += vals.get(k, dest, k, size - k);
    }
    return size;
  }

```



# 4 子类 DeltaPackedLongValues

存储差值的子类。这个子类存储的非原始数据，而是存储与这批数据最小值的差值。这样就可以使存储的值更小。从而提高压缩比例



```java
    void pack(long[] values, int numValues, int block, float acceptableOverheadRatio) {
      long min = values[0];
      for (int i = 1; i < numValues; ++i) {
        min = Math.min(min, values[i]);
      }
      for (int i = 0; i < numValues; ++i) {
        values[i] -= min;
      }
      super.pack(values, numValues, block, acceptableOverheadRatio);
      mins[block] = min;
    }
```

这里我们看到传给父类的 values 是与最小值min的差值。

对应的在获取值时，也是将差值进行添加

```java
  int decodeBlock(int block, long[] dest) {
    final int count = super.decodeBlock(block, dest);
    final long min = mins[block];
    for (int i = 0; i < count; ++i) {
      dest[i] += min;
    }
    return count;
  }
```

