# 1. timed

将每个元素，封装到Timed对象中，让下游可以感知到当前时间。

```java

    Flux.just(1, 3, 0, 4, 2,1)
                .timed()
                .map(reactor.core.publisher.Timed::timestamp)
                .subscribe(System.out::println);
```

# 2. elapsed

记录自调用subscribe到执行完成后的时间

```java

        Flux.just(1, 3, 0, 4, 2,1)
                .elapsed()
                .map(s ->{
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    return s;
                })
                .subscribe(System.out::println);
```


# 3. timestamp

获取当前时间戳