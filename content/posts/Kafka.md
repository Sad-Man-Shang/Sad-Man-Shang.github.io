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



