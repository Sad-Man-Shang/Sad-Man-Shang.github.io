---
title: "Kafka"
date: 2023-09-05T02:08:03+08:00
draft: false
---

三张图了解kafka

整体架构

![image-20230905020852300](/Users/shengquan/my_website/content/posts/Kafka.assets/image-20230905020852300.png)



主题、分区、备份

![image-20230905020929009](/Users/shengquan/my_website/content/posts/Kafka.assets/image-20230905020929009.png)



两个线程

![image-20230905020940684](/Users/shengquan/my_website/content/posts/Kafka.assets/image-20230905020940684.png)

集群模式下，多个一个broker中可能有某个leader也可能有某个broker



![image-20230914213337166](/Users/shengquan/my_website/content/posts/Kafka.assets/image-20230914213337166.png)



关键点：

拦截器如何创建、使用？（在生产者，继承一个接口，Producerinterceptor，拦截器的添加需要使用List<\String>进行添加，string是全限定名

## Producerinterceptor详解

```java
// ProducerRecord中的内容不可被更改，因为有主题、分区等属性，随意修改可能不安全，所以需要重新new一个
// 不要修改topic、分区、key，修改key会影响分区以及压缩日志的功能
// 修改value不应该影响value的序列化
public ProducerRecord onSend(ProducerRecord producerRecord);
// 消息发送前以及发送失败都会调用，先于callback执行
public void onAcknowledgement(RecordMetadata recordMetadata， Exception e);
// 清理工作
public void close();

public void configure(Map<String，?> map) [

```



## 如何对发送的消息进行转化？在生产者的拦截器上进行转化，在onSend中进行实现



## 序列化方法是什么？

json->String的序列化，json->byte[]的序列化，第三方的序列化protobuf格式、avro格式、Thrift格式、json格式配合实现Serializer<>使用



## 如何发送到固定分区？

组装ProducerRecord的时候判断kv是否符合

自定义分区器，实现partitioner，重写partition方法

ProducerRecord中可以指定分区，可以但是不推荐，会让消息组装和逻辑耦合，不太雅观

不设置分区号，就会使用DefaultPartitioner，使用murmur2Hash+topositive计算key的hash，并且%分区数



key为null的情况，default分区去会调用stickyPartitionCache的partion方法进行nullkey的分区操作。查看之前该topic的分区，如果之前没发送过，那就在**可用**的分区中轮训。如果想要将null发送到指定分区，自己创建分区器。



## 如何保证消息发送成功且保证吞吐量？实现Callback，放入send，

```java
public void onCompletion(RecordMetadata recordMetadata, Exception e);
```

消息拦截器的onAcknowledgement方法，只要你发送消息，就能够返回消息的一些offset等信息，只要收到这些信息就说明发送成了。不推荐，因为这样会非常的损耗性能，吞吐量不能够得到保证。

发送消息的send方法，我们已经使用过了，这个send方法有一个`Future<RecordMetadata>`返回值类型，可以接受异步线程的执行结果，我们可以通过future的get方法，但是get是会阻塞的啊。。。

如果不调用send方法返回的future的get方法，就会导致在异常的情况下，无法获取到异常信息，就会导致消息发送的失败，虽然不调用get方法是异步发送消息，有高的吞吐量，但是又不能保证kafka消息发送成功。因此，我们需要添加一个callback接口，当我们的消息有发送失败的时候，这个callback接口就能够起到它的作用，callback接口是异步回调机制。即使使用retry，也要使用callback来为retry兜底



## acks参数如何设置？P11

0:不理会返回，性能最高，记录网站点击量、页面停留时间、视频点击量

1:只要集群首领收到了就收到了

all/-1:所有的都收到才收到





## 发送批次设计

为了提高吞吐量，batch发送

消息累加器中，生产者有32MB的buffer.menory存放消息批次

ProducerBatch的大小batch.size 为16kb（满16kb发送）

linger.ms= 10000，一个ProducerBatch初次收到消息后，需要等待多久（收到消息后10秒内一定发）

调整这两个参数的目的是为了在延迟允许的前提下，减少发送次数



## 8 kafka发送大消息（比batch.size大）要注意什么？

消息在batch.size之内，会复用java.io.Bytebuffer，不然直接发送

直接发送的问题就是要每次都申请消息大小的空间，所以如果频繁发送大消息，不如调高batch.size

另外有max.request.size的默认1mb，超过这个无法发送，需要调整。同时，这个是发送最大值，而不是broker的接收最大值，调整broker的message.max.bytes，同时消费者的fetch.max.bytes

更大的比如10MB，别用Kafka了，用文件传输协议吧。如果一定要用的话：

```
1.
消息拆分并且单线程发送到指定分区，这样确保顺序且在一个分区内，可以拼回去
2.
对消息进行压缩，kafka的生产者支持消息的压缩，通过参数compression.type进行消息压缩。 可以选择snappy，gzip，lz4等压缩形式。确保消息能够最小的发送。
```



## 9 retry机制

不用：通过callback机制对发送失败的消息进行监听，然后再次进行发送（失败的消息保存到db中，日志中）

用：kafka 通过retries 参数控制重试的次数，通过retry.backoff.ms控制重试次数之间的时间间隔。你们这个retry.backoff.ms这个时间间隔设置的是多大啊？（第二次连击）我们的时间间隔是500ms，之所以设置为500ms，是因为我们kafka中topic分区的副本首领选举的整个过程是500ms以内完成的。kafka的retry这机制，不是每次都retry的，如果收到了这样的error：消息大小超过了request.max.size 或者超过了message.max.bytes 类似这样的错误，kafka是不会选择重试的，因为这种错误是无法通过重试而成功的。如果是因为网络延迟、网络抖动啊、分区的一系列暂时不可用啊，这种错误kafka认为有可能在重试的过程中成功。



## 10 顺序消费

打到同一主题同一分区，并且对这个分区的数据进行消费

消息失败，不用retry，使用callback，可能会乱序：1、2、3、1失败、1重试

```
max.in.flight.requests.per.connection，这个参数的意义是，一个kafka的生产者，同时允许多少个未发送成功的消息存在。如果max.in.flight.requests.per.connection参数 = 1，那么只要有一条消息发送未成功并且一直在重试，那么其他的消息是不能再次发送的，1的意思就是，发一条，咱们就收到1条的结果，这条结果成功也好，失败也好，这都是一个结果，1就代表收到这个消息的发送结果。但是重试就不是了，重试的意思是不认命，结果还没定下来啊。如果max.in.flight.requests.per.connection 的值大于1的，刚才我们发送1,2,3，的场景，就可能出现乱序。因为即便1的发送在重试，那也只是1条信息没有收到结果，max.in.flight.requests.per.connection的值是大于1的，所以2和3还能够继续发送。
所以，在顺序消息发送和消费的场景下，必须要保证max.in.flight.requests.per.connection= 1.

最大在飞请求
```



## 11 消费者和消费者组

![image-20230914204803370](/Users/shengquan/my_website/content/posts/Kafka.assets/image-20230914204803370.png)





## 12 分区再均衡

![image-20230914204756113](/Users/shengquan/my_website/content/posts/Kafka.assets/image-20230914204756113.png)





## 13 消费者的poll仅仅是为了获取数据吗？

```
1. 获取数据，并且poll方法支持阻塞获取；
2. 分区再均衡 （poll方法它就是从分区里获取数据的，它与分区的关系是最为紧密的）
3.群组协调 （高级部分说明）新加入了消费者，消费者会进行 joingroup 操作。
4. 发送心跳。（我们消费者组里边可以有很多消费者，有消费者如果发生异常断连，那么这个消费者就标记为不可用，其他消费者还需要进行该分区的消费，触发一次再均衡。）
5. 提交偏移量，如果消费者的 enable.auto.commit 配置设置为 true（默认值），poll() 方法将定期自动提交最近拉取的消息的偏移量，这意味着这些消息已被消费者处理。

```



## 14 如何进行幂等校验或者数据筛查？

```
1. 我们可以接收到消息后，对消息进行判断，筛查以及幂等校验。（不建议使用这种方式。）

2. 自定义消费者拦截器。implements ConsumerInterceptor接口，里边有onConsume方法，这个方法是poll方法返回之前被调用的。
3. onCommit 方法是对 offset偏移量（消费者消费数据后，会提交一个offset偏移量，kafka根据这个偏移量可以确定消息是否被消费了）提交后调用的方法。 我们可以从该方法中获取一些偏移量的信息，不过，没有额外的需求，这部分可以保持为空方法体。

如果一条消息有生成的 timestamp。我们可以用拦截器消息过期丢弃的操作，用此来模拟 类似于rabbitmq的ttl机制。

```

## 15消费者提交偏移量的经验

```
1. 自动提交 enable.auto.commit = true
	提交异常会导致重复消费
2. 手动提交consumer.commitSync();
	sync，同步提交，会阻塞
3. 异步提交
	consumer.commitASync();
	防止同步阻塞导致吞吐量降低
4. 同步+异步
```

