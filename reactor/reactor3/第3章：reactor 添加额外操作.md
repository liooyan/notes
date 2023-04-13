# 1 doOnNext

在流中添加一个没有返回值的方法，每个元素向下传播时触发这个方法，类似与java8的foreach

```java
 Flux.just("1q", "2q", "3q")
                .doOnNext(System.out::println)
                .subscribe(System.out::println);

```

# 2 doOnComplete 与 doOnSuccess

在当前流执行完成后调用一次

```java
  Flux<String> stringFlux = Flux.just("1q", "2q", "3q")
                .doOnComplete(() -> System.out.println(2134));
        stringFlux.subscribe(System.out::println);
        stringFlux.subscribe(System.out::println);

```

# 3 doOnError

捕获异常操作，如果在流的中途抛出，则后续元素将不再执行

```java
        Flux.just(1, 3, 0, 4, 2)
                .map(s -> 1 / s)
                .doOnError(Throwable::printStackTrace)
                .subscribe(System.out::println);
```

# 4 doOnCancel

流被取消时，触发的行为

# 5 doFirst

在流执行前触发

```java


        Flux.just(1, 3, 0, 4, 2)
                .doFirst(()-> System.out.println("qqq"))
                .subscribe(System.out::println);
```

# 6 doOnSubscribe

在onSubscribe 执行时触发

# 7 doFinally

在执行完成时触发

# 8 log
日志行，但对于结果没有特别理解

```java

Flux.just(1, 3, 0, 4, 2)
                .log("测试", Level.WARNING)
                .subscribe(System.out::println);

```