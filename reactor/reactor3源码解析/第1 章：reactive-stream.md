# 1. 说明

当我们引入reactor的依赖时

```xml
  		<dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-core</artifactId>
        </dependency>
```



发现除了reactor-core外，还依赖一个reactive-stream包。而在这个包中，只有4个接口

![image-20211105154342305](../../../z-image/image-20211105154342305.png)

而这几个接口就是定义了非阻塞背压的异步处理流程标准。我们下面详细说明





# 2. reactive-stram解决了什么问题

我们知道reactor是基于发布/订阅模式，一个生成者，一个消费者。一个负责生产数据，一个负责消费数据，如果在一个线程中，不会有什么问题，但如果是分布式或者异步系统中，就会出现一下问题：

- 生产者无法感知消费者的状态，不知道消费者到底是繁忙状态还是空闲状态，是否有能力去消费更多的数据。

所以这时，就需要引入back-pressure模式，背压模式。由消费者通知发送者是否需要发送数据，和停止发送数据。



而reactive-stream就是一个背压模式的标注，它只定义了协议的接口，具体实现由具体框架来实现。



# 3  reactive-stram 接口定义



## 3.1 Publisher

生产者

```JAVA
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}
```



Publisher就是用来生成消息的。它定义了一个subscribe方法，传入一个Subscriber。这个方法用来将Publisher和Subscriber进行连接。

一个Publisher可以连接多个Subscriber。



## 3.2 Subscriber

消费者

```java
public interface Subscriber<T> {
    public void onSubscribe(Subscription s);
    public void onNext(T t);
    public void onError(Throwable t);
    public void onComplete();
}
```



Subscriber就是消息的接收者。

在Publisher和Subscriber建立连接的时候会触发onSubscribe(Subscription s)方法。

当生产者向消费者发送数据时调用onNext

在发生异常或者结束时会触发onError(Throwable t)或者onComplete()方法。



## 3.3 Subscription



```java
public interface Subscription {
    public void request(long n);
    public void cancel();
}
```



Publisher每次调用subscribe建立连接，都会创建一个新的Subscription，Subscription和subscriber是一一对应的。

我们看到在Subscriber的onSubscribe方法中通过参数传给了Subscriber。

这样Subscriber在需要Publisher发送数据时，就可以通过调用Subscription的request方法告知Publisher发送指定数量的数据。

当Subscriber不需要数据时，通过调用cancel，停止数据发送。



# 4 总结

reactive stream的出现有效的解决了异步系统中的背压问题。只不过reactive stream只是一个接口标准或者说是一种协议，具体的实现还需要自己去实现。



![image-20211105165952936](../../../z-image/image-20211105165952936.png)



