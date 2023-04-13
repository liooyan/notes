# 1 一对一的转换
## 1.1 map
与java stream相似的操作

```java

     Flux<String> strJust = Flux.just("1q", "2q", "3q");
        strJust.map(s ->s+":qqq")
                .subscribe(System.out::println);

```

## 1.2 cast 将每个元素转化为需要的类型


```java

  static class A {

    }

    static class B extends A {
        public int pr() {
            return 11;
        }
    }

    public static void main(String[] args) {

        Flux<A> just = Flux.just(new B(), new B());
        just.cast(B.class)
                .subscribe(s -> System.out.println(s.pr()));

    }

```

## 1.3 index 添加元素顺序的下标

```java

 Flux<String> strJust = Flux.just("1q", "2q", "3q");
        strJust.index()
                .map(s -> s.getT1() + "_" + s.getT2() + ":qqq")
                .subscribe(System.out::println);


        strJust.index((index, value) -> index + "_" + value + ":qqq" )
                .subscribe(System.out::println);

```


# 2 一对n的转换
## 2.1  flatMap
与java相同

```java

        Flux<String> strJust = Flux.just("1q", "2q", "3q");
        strJust.flatMap(s -> Flux.just(s, "1"))
                .subscribe(System.out::println);

```


## 2.1  handle

与map 相似
```JAVA

Flux<String> strJust = Flux.just("1q", "2q", "3q");
strJust.handle((value, sink) -> {    sink.next(value+"1");})
        .subscribe(System.out::println);


```

# 3 在序列开始或结束是添加元素


## 3.1 startWith 开始


```java

        Flux<String> strJust = Flux.just("1q", "2q", "3q");
        strJust.startWith("1","2","3")
                .subscribe(System.out::println);
```

## 3.2 concatWithValues 结束

```java

Flux<String> strJust = Flux.just("1q", "2q", "3q");
strJust.concatWithValues("1","2","3")       
        .subscribe(System.out::println);


```


# 4 聚合操作

## 4.1 collectList

将所有元素合并为一个list

```java


   Flux<String> strJust = Flux.just("1q", "2q", "3q");
        List<String> block = strJust.collectList()
                .block();
        System.out.println(block);
        
        
        strJust = Flux.just("2q", "1q", "3q");
         block = strJust.collectSortedList()
                .block();
        System.out.println(block);
```


## 4.2 collectMap
将所有元素合并为一个map


```java


        Flux<String> strJust = Flux.just("1q", "2q", "3q");
        Map<String, String> block = strJust.collectMap(Function.identity())
                .block();
        System.out.println(block);


         strJust = Flux.just("1q", "2q", "3q", "2q", "3q");
        Map<String, Collection<String>> block2 = strJust.collectMultimap(Function.identity())
                .block();
        System.out.println(block2);

```


## 4.3 collect

使用java8 StreamApi的Collector 转换流


```java

        Flux<String> strJust = Flux.just("1q", "2q", "3q");
        List<String> block = strJust.collect(Collectors.toList())
                .block();
        System.out.println(block);


```

## 4.4 count 

计数

```java

        Flux<String> strJust = Flux.just("1q", "2q", "3q");
        Long block = strJust.count().block();
        System.out.println(block);

```

## 4.5 reduce

  reduce 和 reduceWith 操作符对流中包含的所有元素进行累积操作，得到一个包含计算结果的 Mono 序列。


```java

        Flux<String> strJust = Flux.just("1q", "2q", "3q");
        String block = strJust.reduce((a, b) -> a + "_" + b).block();

        System.out.println(block);

        String block1 = strJust.reduceWith(() -> "-1", (a, b) -> a + "_" + b).block();

        System.out.println(block1);


```


## 4.6 scan

与reduce 相同，将元素进行累加，但返回的是Flux对象，每个元素是累加过程中的每一步结果

```java


    Flux<String> strJust = Flux.just("1q", "2q", "3q");
        Flux<String> block = strJust.scan((a, b) -> a + "_" + b);
        block.subscribe(System.out::println);
```

## 4.7 all

通过Predicate对象，遍历每个元素，返回一个boolean，如果所有的都为true。则最后返回ture。否则为false。

```java

    Flux<String> strJust = Flux.just("1q", "2q", "3q");
        Boolean block = strJust.all(a->true).block();

        System.out.println(block);
```


## 4.8 any

与all 相似，如果有一个为true，返回true。当所有都为false是，返回false。

```java

    Flux<String> strJust = Flux.just("1q", "2q", "3q");
        Boolean block = strJust.any(a->true).block();

        System.out.println(block);
```


## 4.9 hasElements 与 hasElement

hasElements 判断当前流中是否有元素

hasElement 判断当前流中是否有指定的元素





# 5  合并操作


## 5.1 concatWith

合并两个流

```java

        Flux<String> strJust = Flux.just("1q", "2q", "3q");
        Flux<String> strJust2 = Flux.just("1q", "2q", "3q");
        Flux<String> stringFlux = strJust.concatWith(strJust2);
        stringFlux.subscribe(System.out::println);

```


## 5.2 mergeSequential

合并两个流,将其元素穿插合并，而不是之间添加在末尾


## 5.3 zip

对于不同元素的两个流，将其合并为一个一个的 元素对

```java


        Flux<String> strJust = Flux.just("1q", "2q", "3q");
        Flux<Integer> strJust2 = Flux.just(1,2,3,5);
        Flux<Tuple2<String, Integer>> stringFlux = strJust.zipWith(strJust2);
        stringFlux.subscribe(System.out::println);
```


## 5.4 switchMap

结果与flatMap相同，具体细节在看源码时理解



# 6 复制操作

## 6.1 repeat

无限运行现有流，或者指定循环次数

```java

        Flux<String> strJust = Flux.just("1q", "2q", "3q");
        Flux<String> repeat = strJust.repeat(1000L);
        repeat.subscribe(System.out::println);
```


# 7 空的流操作

## 7.1 defaultIfEmpty
为空的流添加一个默认元素


```java

        Flux<String> strJust = Flux.just()
                .cast(String.class)
                .defaultIfEmpty("qqqq");

        strJust.subscribe(System.out::println);

```


## 7.2 switchIfEmpty
当指定的流为空的时候，使用指定的流进行替代


```java


        Flux.just()
                .cast(String.class)
                .switchIfEmpty(Flux.just("qqqq", "qqqq2"))
                .subscribe(System.out::println);
```

# 8 丢弃不需要的元素

## 8.1 ignoreElements

将流转换为一个空的Mono对象

```java

        Flux.just("qqqq", "qqqq2")
                .map(s ->{
                    System.out.println(111);
                    return s;
                })
                .ignoreElements()
                .subscribe(System.out::println);
```


## 8.2 then

使用一个新的元素替代我们之前的流


```java

        Flux.just("qqqq", "qqqq2")
                .map(s ->{
                    System.out.println(111);
                    return s;
                })
                .then()
                .subscribe(System.out::println);
```

## 8.3 delayUntil

//TODO


# 9 递归 操作

## 9.1 expand


广度优先遍历策略递归



## 9.2  expandDeep


深度优先的遍历顺序递归


## 9.3 测试用例


```JAVA

  public static void main(String[] args) {
        Node root = new Node("0");
        Node node1_1 = new Node("1-1");
        Node node1_2 = new Node("1-2");
        Node node2_1 = new Node("2_1");
        Node node2_2 = new Node("2_2");
        Node node2_3 = new Node("2_3");
        Node node2_4 = new Node("2_4");
        root.nodeRefs.add(node1_1);
        root.nodeRefs.add(node1_2);
        node1_1.nodeRefs.add(node2_1);
        node1_1.nodeRefs.add(node2_2);
        node1_2.nodeRefs.add(node2_3);
        node1_2.nodeRefs.add(node2_4);


        Mono.just(root)
                .expand(node ->  Flux.fromIterable(node.nodeRefs) )
                .subscribe(System.out::println);


        Mono.just(root)
                .expandDeep(node ->  Flux.fromIterable(node.nodeRefs) )
                .subscribe(System.out::println);
    }

    static class Node {


        public Node(String id) {
            this.id = id;
        }

        String id;
        List<Node> nodeRefs = new ArrayList<>();

        @Override
        public String toString() {
            return "Node{" +
                    "id='" + id + '\'' +
                    '}';
        }
    }


```