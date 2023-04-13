## 1 filter
与java stream相同
```java


        Flux.just(1, 3, 0, 4, 2)
                .filter(s -> s == 0)
                .subscribe(System.out::println);


        Flux.just(1, 3, 0, 4, 2)
                .filterWhen(s -> s >=2 ? Mono.just(true): Mono.just(false))
                .subscribe(System.out::println);
```


## 2 ofType
过滤符合指定类型的元素

```java


        Flux.just(1,"23",123L,42)
                .ofType(Integer.class)
                .subscribe(System.out::println);
```


## 3 distinct


去重，或者按照指定的key进行删除
```java

        Flux.just(1, 3, 0, 4, 2,1)
                .distinct()
                .subscribe(System.out::println);



        Flux.just(1, 3, 0, 4, 2,1)
                .distinct((a) -> a-1 )
                .subscribe(System.out::println);
```

## 4 take

获取前n个元素

```java

   Flux.just(1, 3, 0, 4, 2,1)
                .take(3)
                .subscribe(System.out::println);
```


takeLast 获取后n个元素

```java

  Flux.just(1, 3, 0, 4, 2,1)
                .takeLast(3)
                .subscribe(System.out::println);
```

## 5 skip

跳过前n个元素
```java

  Flux.just(1, 3, 0, 4, 2,1)
                .skip(3)
                .subscribe(System.out::println);
```