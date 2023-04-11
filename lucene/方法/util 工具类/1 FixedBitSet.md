# 1 FixedBitSet

FixedBitSet 使用固定长度的long数组，存储 较连续、无重复的数据集。

一般用于存储文档id， 它通过使用long数组的每一个bit位来描述当前数据集是否存储此文档id。





# 2 测试用例

```java
    public static void main(String[] args)
    {
        org.apache.lucene.util.FixedBitSet fixedBitSet = new org.apache.lucene.util.FixedBitSet(300);
        fixedBitSet.set(1);
        fixedBitSet.set(2);
        fixedBitSet.set(3);
        fixedBitSet.set(4);
        fixedBitSet.set(5);
        fixedBitSet.set(6);
        
        boolean have7 = fixedBitSet.get(7);
        System.out.println(have7);
    }
```





# 3 构造函数

```java
 public FixedBitSet(int numBits) {
    this.numBits = numBits;
    bits = new long[bits2words(numBits)];
    numWords = bits.length;
  }
```

创建出够我们存储的 long数组。 比如我们要存储300个文档id，每个long 可以存储64个数据，则创建出的数组长度为5



# 4 主要方法

## 4.1 set 添加元素

```java
  public void set(int index) {
    assert index >= 0 && index < numBits: "index=" + index + ", numBits=" + numBits;
    int wordNum = index >> 6;      // div 64
    long bitmask = 1L << index;
    bits[wordNum] |= bitmask;
  }
```

添加元素就是将对应下标的bit位置为 1



## 4.2 get 获取元素

```java
  public boolean get(int index) {
    assert index >= 0 && index < numBits: "index=" + index + ", numBits=" + numBits;
    int i = index >> 6;               // div 64
    // signed shift will keep a negative index and force an
    // array-index-out-of-bounds-exception, removing the need for an explicit check.
    long bitmask = 1L << index;
    return (bits[i] & bitmask) != 0;
  }
```

与set对应，判断对应bit位的值是否为1



## 4.3 ensureCapacity 扩容并返回一个新的集合

```java

  public static FixedBitSet ensureCapacity(FixedBitSet bits, int numBits) {
     //首先判断，是否需要扩容
    if (numBits < bits.numBits) {
      return bits;
    } else {
      // Depends on the ghost bits being clear!
      // (Otherwise, they may become visible in the new instance)
      int numWords = bits2words(numBits);
      long[] arr = bits.getBits();
      if (numWords >= arr.length) {
        arr = ArrayUtil.grow(arr, numWords + 1);
      }
      // 创建一个新的集合，大小为原来 2^6,并将数据复制到新的集合
      return new FixedBitSet(arr, arr.length << 6);
    }
  }
```







