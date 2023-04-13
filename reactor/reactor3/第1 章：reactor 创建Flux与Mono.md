# 1 just 

# 1.1 一组元素创建

```java

@SafeVarargs 
public static <T>  Flux <T> just(T... data)


public static <T> Mono<T> just(T data)

```


由一组元素创建Flux

测试用例：
```java
        Flux<String> strJust = Flux.just("1q", "2q", "3q");
        strJust.subscribe(System.out::println);

        Flux<Integer> intJust = Flux.just(1,2,3,4);
        intJust.subscribe(System.out::println);

        Mono<String> mono = Mono.just("1Mono");
        mono.subscribe(System.out::println);

```


# 1.2 justOrEmpty 由 Optional 创建

对于Mono可以使用justOrEmpty，过滤null元素，同时，也可以传入Optional对象。如果为empty，则会被过滤掉。
测试用例：

```java

 public static void main(String[] args) {
        Optional<String>  optionalS =  Optional.of("test");
        Mono<String> mono = Mono.justOrEmpty(optionalS);
        mono.subscribe(System.out::println);


        mono = Mono.justOrEmpty(null);
        mono.subscribe(System.out::println);


        mono = Mono.justOrEmpty(Optional.empty());
        mono.subscribe(System.out::println);


        mono = Mono.just(null);
        mono.subscribe(System.out::println);

    }

```


我们可以看到如果使用justOrEmpty则会过滤掉null元素，如果使用just，对于null则会抛出异常。



# 2 由数组或 iterates  创建



## 2.1 fromArray 由数组创建

```java
public static <T>  Flux <T> fromArray(T[] array)
```

## 2.2 fromIterable 由Iterable 创建

```java
public static <T>  Flux <T> fromIterable( Iterable <? extends T> it)
```


## 2.3 fromStream 由Stream 创建

```java
public static <T>  Flux <T> fromStream( Stream <? extends T> s)
```

## 2.4 测试用例
```java

  public static void main(String[] args) {
        String[] strs = new String[]{"a","b","c","d"};
        Flux<String> stringFlux = Flux.fromArray(strs);
        stringFlux.subscribe(System.out::println);

       List<String> str =  new ArrayList<>();
        str.add("listA");
        str.add("listB");
        str.add("listC");
        str.add("listD");
        stringFlux = Flux.fromStream(str.stream());
        stringFlux.subscribe(System.out::println);


        stringFlux = Flux.fromIterable(str);
        stringFlux.subscribe(System.out::println);

    }

```


# 3 由Supplier 提供


通过Supplier加载的都是延迟加载，只有当我们调用subscribe时，才会加载数据
## 3.1 fromSupplier 


由一个方法提供需要的元素并创建Mono对象。测试用例：

```java

Mono<String> monoStr = Mono.fromSupplier(() -> "qws");
monoStr.subscribe(System.out::println);


```


如果我们多次调用subscribe方法则会多次通过Supplier 创建不同的元素。测试用例如下：

```java
    static int anInt = 0;
    public static void main(String[] args) {

      

        Mono<Integer> mono = Mono.fromSupplier(() -> anInt++);

        mono.subscribe(System.out::println);
        mono.subscribe(System.out::println);
        mono.subscribe(System.out::println);


    }

```

## 3.2 fromCallable, fromRunnable

fromCallable与fromSupplier 相似，接收一个fromCallable对象，并根据返回值创建元素。

fromRunnable 返回的是一个空Mono对象，当我们调用起subscribe方法时，会调用一次Runnable的run方法


测试用例：
```java

        Mono<String> mono = Mono.fromCallable(() -> "12314");
        mono.subscribe(System.out::println);

        mono = Mono.fromRunnable(() -> System.out.println(1234));
        mono.subscribe(System.out::println);
        mono.subscribe(System.out::println);
```



## 3.3 fromFuture

//TODO 目前还没明白

# 4 empty 创建一个空的流

Mono<String> empty = Mono.empty();



# 5 error 创建一个会立即报错的流

error创建一个Mono.当我们调用subscribe时会报异常

```java
Mono<Object> error = Mono.error(Exception::new);
error.subscribe();



```
# 6 defer 创建一个延迟加载的流

 delay(Duration duration)和 delayMillis(long duration)：创建一个 Mono 序列，在指定的延迟时间之后，产生数字 0 作为唯一值。

```java

        long start = System.currentTimeMillis();
        Disposable disposable = Mono.delay(Duration.ofSeconds(2)).subscribe(n -> {
            System.out.println("生产数据源："+ n);
            System.out.println("当前线程ID："+ Thread.currentThread().getId() + ",生产到消费耗时："+ (System.currentTimeMillis() - start));
        });
        System.out.println("主线程"+ Thread.currentThread().getId() + "耗时："+ (System.currentTimeMillis() - start));
        while(!disposable.isDisposed()) {
        }

```

# 7 using 

//TODO

# 8 generate

```java


 public static void main(String[] args) {

        Flux.generate(sink -> {
            sink.next("Hello");
            sink.complete();
        }).subscribe(System.out::println);


        final Random random = new Random();
        Flux.generate(ArrayList::new, (list, sink) -> {
            int value = random.nextInt(100);
            list.add(sink);
            sink.next(random.nextInt(100));
            if (list.size() == 10) sink.complete();
            return list;
        }).subscribe(System.out::println);


        Flux.generate(() -> 0, (num, sink) -> {
            if (num == 1) sink.complete();
            sink.next(num += 1);
            return num;
        }).subscribe(System.out::println);

    }

```

# 9 create
create()方法与 generate()方法的不同之处在于所使用的是 FluxSink 对象。FluxSink 支持同步和异步的消息产生，并且可以在一次调用中产生多个元素。
```JAVA

        Flux.create(sink -> {
            for (int i = 0; i < 10; i++) {
                sink.next(i);
            }
            sink.complete();
        }).subscribe(System.out::println);



```