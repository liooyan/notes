# 1 创建异常
通过error方法抛出一个新的异常

```java
     Flux.just(1, 3, 0, 4, 2)
                .error(Exception::new)
                .subscribe(System.out::println);


```


# 2 异常捕获

## 2.1 onErrorReturn

异常时设置返回的默认值,但后续的元素将不会执行

```java
        Flux.just(1, 3, 0, 4, 2)
                .map(s -> 1/s)
                .onErrorReturn(-99)
                .subscribe(System.out::println);

```


## 2.2 onErrorResume

异常时，跳转到另一个流中,同样原来的后续元素将不再执行

```java

        Flux.just(1, 3, 0, 4, 2)
                .map(s -> 1 / s)
                .onErrorResume(s -> Flux.just(1, 3, 0, 4, 2))
                .subscribe(System.out::println);
```

##  2.3 onErrorMap 

将异常包装为一个新的异常

```java

        Flux.just(1, 3, 0, 4, 2)
                .map(s -> 1 / s)
                .onErrorMap(e ->new NullPointerException())
                .subscribe(System.out::println);

```

## 2.4 doFinally

finally块



# 3 retry 重试

```java

        Flux.just(1, 3, 0, 4, 2)
                .map(s -> 1 / s)
                .retry(5)
                .subscribe(System.out::println);
```