# 1 背景

本章将通过一个简单的测试用例，说明reactor源码的基本运行流程



# 2 测试用例

```java
Flux.just("1q", "2q", "3q")
                .map(s -> s + ":qqq")
                .filter(s -> s.equals("2q:qqq"))
                .subscribe(System.out::println);
```



# 3、源码分析

## 3.1 just

我们先看一下Flux的just方法

```java
@SafeVarargs
public static <T> Flux<T> just(T... data) {
   return fromArray(data);
}
```

```java
public static <T> Flux<T> fromArray(T[] array) {
   if (array.length == 0) {
      return empty();
   }
   if (array.length == 1) {
      return just(array[0]);
   }
   return onAssembly(new FluxArray<>(array));
}
```

​	这里我们看到其实最后创建的是FluxArray，而它的构造函数如下：

```java
final T[] array;

@SafeVarargs
public FluxArray(T... array) {
   this.array = Objects.requireNonNull(array, "array");
}
```

这里将我们传入的数组保存到了成员变量中。

## 3.2 map

上面我们分析了通过just创建Flux，下面我们看一下map都做了什么

```java
public final <V> Flux<V> map(Function<? super T, ? extends V> mapper) {
   if (this instanceof Fuseable) {
      return onAssembly(new FluxMapFuseable<>(this, mapper));
   }
   return onAssembly(new FluxMap<>(this, mapper));
}
```

`onAssembly` 是一个扩展机制，因为实例中并没有使用，所以可简单理解为将入参直接返回。这里我们先忽略onAssembly。

我们看到分为了两类，就是返回的值分为了 `Fuseable` 和非 `Fuseable` 两种。Fuseable 的意思就是可融合的。我理解就是 `Flux` 表示的是一个类似集合的概念，有一些集合类型可以将多个元素融合在一起，打包处理，而不必每个元素都一步步的处理，从而提高了效率。因为 `FluxArray` 中的数据一开始就都准备好了，因此可以打包处理，因此就是 `Fuseable`。

参考：https://www.jianshu.com/p/df395eb28f69

这里参数有两个 this和mapper。this也就是我们上一步创建的FluxArray对象，而mapper对象也就是我们传入的`s -> s + ":qqq"`我们跟进构造函数：

```java
final Function<? super T, ? extends R> mapper;


FluxMapFuseable(Flux<? extends T> source,
      Function<? super T, ? extends R> mapper) {
   super(source);
   this.mapper = Objects.requireNonNull(mapper, "mapper");
}
```

我们看到这里我们将mapper先保存了起来，这也就是为什么reator的方法不会立即触发而是异步触发的原因。

而`source`则是传到了父类，它最后是在父类`FluxO[erator`中保存

```java
protected final Flux<? extends I> source;


protected FluxOperator(Flux<? extends I> source) {
   this.source = Objects.requireNonNull(source);
}
```



# 3.3 filter

filter与map相似，代码如下：

```java
public final Flux<T> filter(Predicate<? super T> p) {
   if (this instanceof Fuseable) {
      return onAssembly(new FluxFilterFuseable<>(this, p));
   }
   return onAssembly(new FluxFilter<>(this, p));
}
```

这里不再重复



# 3.4 声明阶段总结

通过上面3个步骤，我们看到我们从数据源到不同的操作算子的一个声明阶段。

而在这个阶段中，不会直接触发计算，而是通过一种 装饰设计模式，将其封装。

上面的3个类`FluxArray`,`FluxMap`, `FluxFilter`都继承自Flux。而每一次的方法调用都是都是创建一个新的对象，将原来的对象保存在这个新的对象中，并将新的对象返回给使用这。

也就是说,在最后我们持有的是`FluxFilter`而它持有`FluxMap`而`FluxMap`持有`FluxArray`.



# 3.5 subscribe

在声明完成后，我们看一下subscribe方法。

```java
public final Disposable subscribe(
      @Nullable Consumer<? super T> consumer,
      @Nullable Consumer<? super Throwable> errorConsumer,
      @Nullable Runnable completeConsumer,
      @Nullable Context initialContext) {
   return subscribeWith(new LambdaSubscriber<>(consumer, errorConsumer,
         completeConsumer,
         null,
         initialContext));
}
```

这里我们看到我们传入的函数被封装到`LambdaSubscriber`对象中，而`LambdaSubscriber`	对象中，也就是消费者。

我们继续看subscribeWith方法

```java
public final <E extends Subscriber<? super T>> E subscribeWith(E subscriber) {
   subscribe(subscriber);
   return subscriber;
}
```

```java
@Override
@SuppressWarnings("unchecked")
public final void subscribe(Subscriber<? super T> actual) {
  //扩展机制,暂时忽略
   CorePublisher publisher = Operators.onLastAssembly(this);
   CoreSubscriber subscriber = Operators.toCoreSubscriber(actual);

   try {
     	//判断是否为运算符，上面的3个类中，除了FluxArray外，都是。
      if (publisher instanceof OptimizableOperator) {
         OptimizableOperator operator = (OptimizableOperator) publisher;
        //循环
         while (true) {
           //获取当前运行符的subscriber
            subscriber = operator.subscribeOrReturn(subscriber);
            if (subscriber == null) {
               // null means "I will subscribe myself", returning...
               return;
            }
           //返回下一个OptimizableSource，默认情况下与operator.source相同
            OptimizableOperator newSource = operator.nextOptimizableSource();
            if (newSource == null) {
               publisher = operator.source();
               break;
            }
            operator = newSource;
         }
      }
			//触发subscribe
      publisher.subscribe(subscriber);
   }
   catch (Throwable e) {
      Operators.reportThrowInSubscribe(subscriber, e);
      return;
   }
}
```



在上一步中我们持有的是`FluxFilter`对象。他们通过`FluxFilter` ->`FluxMap`  ->  `FluxArray` 相互持有的关系。

而我们知道正在的数据生产商是最里面的FluxFilter。所有这里我们看到通过一个循环上级遍历，最终找到了正在的数据持有者publisher，而最后也就是调用了它的subscribe ，从而完成了数据的订阅。



但我们也map、filter它们其实不应该是数据的生产者，他们应该与我们最后的方法以前组成subscriber订阅者。这里就要说到`operator.subscribeOrReturn(subscriber)`方法，我们具体看一下map的方法

```java
@Override
@SuppressWarnings("unchecked")
public CoreSubscriber<? super T> subscribeOrReturn(CoreSubscriber<? super R> actual) {
   if (actual instanceof ConditionalSubscriber) {
      ConditionalSubscriber<? super R> cs = (ConditionalSubscriber<? super R>) actual;
      return new MapFuseableConditionalSubscriber<>(cs, mapper);
   }
   return new MapFuseableSubscriber<>(actual, mapper);
}
```

我们看到它最后创建了它的子类对象MapFuseableSubscriber，而它就是一个订阅者CoreSubscriber。而它需要一个actual参数，也就它内层Flux产生的subscriber。到最后传给publisher的subscriber就是map产生的subscriber。



最开始产生的Flux链条是`FluxFilter` ->`FluxMap`  ->  `FluxArray`  。这与我们代码中的顺序是相反的。当我们通过循环，在此生成subscriber链条时，刚好顺序会被反过来，反而形成了我们代码的顺序，这样在代码执行时刚好是我们原来的顺序，再加上最后触发的subscriber。组成链条如下：

`MapFuseableSubscriber` ->`FilterFuseableSubscriber`    ->`LambdaSubscriber` 



我们继续看FluxArray的subscribe方法



```java
@Override
public void subscribe(CoreSubscriber<? super T> actual) {
   subscribe(actual, array);
}
```

```java
public static <T> void subscribe(CoreSubscriber<? super T> s, T[] array) {
   if (array.length == 0) {
      Operators.complete(s);
      return;
   }
   if (s instanceof ConditionalSubscriber) {
      s.onSubscribe(new ArrayConditionalSubscription<>((ConditionalSubscriber<? super T>) s, array));
   }
   else {
      s.onSubscribe(new ArraySubscription<>(s, array));
   }
}
```

这里我们看到与我们分析reactive-sream框架预想的一样。当订阅完成后会触发订阅者的onSubscribe方法，并且为当前订阅事件生成自己的Subscription对象。其中s就是我们上面说明的Subscriber处理链条



## 3.6 onSubscribe

通过上面的处理链条。我们知道最开始调用的是MapFuseableSubscriber方法，我们知道它只是一个算子，所以onSubscribe是直接调用下一个链条：

```java
@SuppressWarnings("unchecked")
@Override
public void onSubscribe(Subscription s) {
   if (Operators.validate(this.s, s)) {
      this.s = (QueueSubscription<T>) s;
      actual.onSubscribe(this);
   }
}
```

这个直到最后的一个

```java
@Override
public final void onSubscribe(Subscription s) {
   if (Operators.validate(subscription, s)) {
      this.subscription = s;
      if (subscriptionConsumer != null) {
         try {
            subscriptionConsumer.accept(s);
         }
         catch (Throwable t) {
            Exceptions.throwIfFatal(t);
            s.cancel();
            onError(t);
         }
      }
      else {
         s.request(Long.MAX_VALUE);
      }
   }
}
```

这里我们看到了直接调用了Subscription的request。应为当前是通过数组创建，所以这里就是直接获取了全部的数据。



## 3.7 request

前面我们看到完成onSubscribe后，就调用了request方法，获取数据。我们返回FluxArray的request方法



```java
void fastPath() {
   final T[] a = array;
   final int len = a.length;
   final Subscriber<? super T> s = actual;

   for (int i = index; i != len; i++) {
      if (cancelled) {
         return;
      }

      T t = a[i];

      if (t == null) {
         s.onError(new NullPointerException("The " + i + "th array element was null"));
         return;
      }

      s.onNext(t);
   }
   if (cancelled) {
      return;
   }
   s.onComplete();
}
```

这里我们看到就是一个for循环，向我们之前创建的处理链条发送数据。直到数据发送完成。而发生数据就是通过onNext方法。并且在发生完成后调用onComplete方法。



# 3.8 onNext

onNext 就是处理每条数据的过程。其实我们已经大概可以猜到每个算子的处理逻辑了：

map就是 调用 我们传入的函数，处理数据，将处理后的结果 通过下一个算子的onNext方法将数据交给一下一个算子。

filter 就是调用我们的函数，如果是true。将数据调用下一个算子，如果是false。就不调用。

最后就是我们自己的`subscribe(System.out::println)`,就是直接执行我们的方法。

这些具体方法我们在后面统一说明。



# 4 总结

我们将上面步骤分为一下几步：

- 声明阶段
- subscribe 阶段
- onSubscribe 阶段
- request 阶段
- 调用阶段