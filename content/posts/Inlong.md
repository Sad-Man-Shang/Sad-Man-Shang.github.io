---
title: "Inlong"
date: 2023-07-12T23:00:26+10:00
draft: false
authors: [Sad_man]
tags: [开源, 大数据, 数据集成]
categories: [开源工作]
series: [开源初探]
series_weight: 2
---

# 模版

项目申请书分为三个部分，技术方案、时间规划、项目经历
技术方案
1.请记住，技术方案越详细越有利于项目导师的理解和认可 ;

2.在技术方案中，学生应该重点展示出自己对开源项目的理解程度，可以从以下几点入手:根据自己的理解和前期项目学习储备来拆解目标开源项目，尽可能描述各个模块的功能:重点对项目发布的任务或需求进行分析，体现自己的思考和思路;具体描述自己的技术方案，比如自己的方案将会添加哪些模块，以及这些模块如何与现有模块通信

3.最好能为自己的方案找到一些可执行的依据，包括但不限于 :其他项目的成功经验;
论文等提供的理论依据





# Inlong

一站式数据集成框架，将不同数据源的数据汇集（批量集成和实时集成）到目标端。

## 架构

![image-20230713005256473](/Users/shengquan/Library/Application Support/typora-user-images/image-20230713005256473.png)

**数据链路**

采集层：从各个数据源采集对应的数据，Agent

汇聚层：接收采集层的数据进行统一编码，做协议转换，大量的链接进行收敛，将采集层的数据进行统一处理，并且上报给缓存层，DataProxy

缓存层：MQ服务，支持TubeMQ，Pulsar，Kafka

分拣层：Sort on Flink，通过业务配置的流向，分拣到具体的服务当中；Sort standalone，非Flink版本； 需要自定义消费逻辑的时候，使用SDK进行消费；对接自己的离线/实时计算。



## 组件

![image-20230713010058964](/Users/shengquan/Library/Application Support/typora-user-images/image-20230713010058964.png)

## Demo

![image-20230713010729918](/Users/shengquan/Library/Application Support/typora-user-images/image-20230713010729918.png)



