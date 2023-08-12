---
title: "GRPC"
date: 2023-07-30T15:53:16+10:00
draft: false
authors: [Sad_man]
tags: [面试]
categories: [rpc]
series: [24届从零开始到找到工作]
series_weight: 1
---

# 一、环境配置

安装

`brew install protobuf`



# 二、

## gRPC 核心的设计思路 

​         1. 网络通信 ---> gRPC自己封装网络通信的部分 提供多种语言的 网络通信的封装 （C Java[Netty] GO)
​         2. 协议    ---> HTTP2 传输数据的时候 二进制数据内容。 支持双向流（双工）连接的多路复用。
​         3. 序列化   ---> 基本文本 JSON  基于二进制 Java原生序列化方式 Thrift二进制的序列化 压缩二级制序列化。
​                         protobuf (Protocol Buffers) google开源一种序列化方式  时间效率和空间效率是JSON的3---5倍。
​                         IDL语言 
​         4. 代理的创建 --->让调用者像调用本地方法那样 去调用远端的服务方法。
​                          stub

3. gRPC 与 ThriftRPC 区别
   共性：支持异构语言的RPC。
   区别：
        1. 网络通信 Thrift TCP   专属协议
                   GRPC   HTTP2
        2. 性能角度 ThriftRPC 性能 高于 gRPC
        3. gRPC 大厂背书（Google),云原生时代 与其他组件合作的顺利。所以gRPC应用更广泛。
   
4. gRPC的好处 
   1. 高效的进行进程间通信。
   2. 支持多种语言 原生支持 C  Go Java实现。C语言版本上扩展 C++ C# NodeJS Python Ruby PHP..
   3. 支持多平台运行 Linux Android IOS MacOS Windows。
   4. gPRC序列化方式采用protobuf，效率高。
   5. 使用Http2协议
   6. 大厂的背书

## http2

1. 回顾 Http1.x协议 
   Http1.0协议  请求响应的模式 短连接协议（无状态协议）  传输数据文本结构     单工 无法实现服务端推送 变相实现推动（客户端轮训的方式） 
   Http1.1协议  请求响应的模式 有限的长连接            升级的方式WebSocket 双工 实现服务器向客户端推送。
   总结Http1.x协议 共性
     1. 传输数据文本格式 可读性好的但是效率差。
     2. 本质上Http1.x协议无法实现双工通信。
     3. 资源的请求。需要发送多次请求，建立多个连接才可以完成。 
2. HTTP2.0协议 
     1. Http2.0协议是一个二进制协议，效率高于Http1.x协议，可读性差。
     2. 可以实现双工通信。
     3. 一个请求 一个连接 可以请求多个数据。【多路复用】
3. Http2.0协议的三个概念
     1. 数据流 stream
     2. 消息   message
     3. 帧    frame    参看图
4. 其他的相关概念
     1. 数据流的优先级，可以通过为不同的stream设置权重，来限制不同流的传输顺序。
     2. 流控 client发送的数据太快了，server处理不过来，通知client暂停数据的发送。



## Protobuf



- 文件格式

  ~~~markdown
  .proto
  
  UserService.proto
  OrderService.proto
  ~~~

- 版本设定

  ~~~markdown
  syntax = "proto3";
  ~~~

- 注释

  ~~~markdown
  1. 单行注释   //
  2. 多行注释   /*    */
  ~~~

- 与Java语言相关的语法

  ~~~markdown
  #后续protobuf生成的java代码 一个源文件还是多个源文件  xx.java
  option java_multiple_files = false; 
  
  #指定protobuf生成的类 放置在哪个包中
  option java_package = "com.example";
  
  #指定的protobuf生成的外部类的名字（管理内部类【内部类才是真正开发使用】）
  option java_outer_classname = "UserServce";
  
  ~~~

- 逻辑包【了解】

  ~~~markdown
  # 对于protobuf对于文件内容的管理
  package xxx;
  ~~~

- 导入，复用其他proto文件中定义的Message

  ~~~markdown
  UserService.proto
  
  OrderService.proto
    import "xxx/UserService.proto";
  ~~~

- 基本类型

  | .proto Type | Notes                                                        | C++ Type | Java Type  | Python Type[2]                       | Go Type  |
  | ----------- | ------------------------------------------------------------ | -------- | ---------- | ------------------------------------ | -------- |
  | double      |                                                              | double   | double     | float                                | *float64 |
  | float       |                                                              | float    | float      | float                                | *float32 |
  | int32       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead. | int32    | int        | int                                  | *int32   |
  | int64       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead. | int64    | long       | int/long[3]                          | *int64   |
  | uint32      | Uses variable-length encoding.                               | uint32   | int[1]     | int/long[3]                          | *uint32  |
  | uint64      | Uses variable-length encoding.                               | uint64   | long[1]    | int/long[3]                          | *uint64  |
  | sint32      | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s. | int32    | int        | int                                  | *int32   |
  | sint64      | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s. | int64    | long       | int/long[3]                          | *int64   |
  | fixed32     | Always four bytes. More efficient than uint32 if values are often greater than 228. | uint32   | int[1]     | int/long[3]                          | *uint32  |
  | fixed64     | Always eight bytes. More efficient than uint64 if values are often greater than 256. | uint64   | long[1]    | int/long[3]                          | *uint64  |
  | sfixed32    | Always four bytes.                                           | int32    | int        | int                                  | *int32   |
  | sfixed64    | Always eight bytes.                                          | int64    | long       | int/long[3]                          | *int64   |
  | bool        |                                                              | bool     | boolean    | bool                                 | *bool    |
  | string      | A string must always contain UTF-8 encoded text.             | string   | String     | unicode (Python 2) or str (Python 3) | *string  |
  | bytes       | May contain any arbitrary sequence of bytes.                 | string   | ByteString | bytes                                | []byte   |

- 枚举，枚举的值必须从0开始

  ~~~protobuf
  enum SEASON{
     SPRING = 0;
     SUMMER = 1;
  
  }
  ~~~

- 消息 Message

  ~~~protobuf
  message LoginRequest {
     string username = 1;
     singular string password = 2;
     int32  age = 3;
  }
  
  编号 从1开始 到2^29-1  注意：19000 - 19999 不能用这个区间内的编号，因为他是protobuf自己保留的。
  
  - singular : 这个字段的值 只能是0个或1个 （默认关键字）  null   "123456"
  - repeated
  
  message Result{
     string content = 1;
     repeated string stutas = 2; //这个字段 返回值 是多个 等价于 Java List Protobuf getStatusList()-->List
  }
  
  // 可以定义多个消息 
  message LoginRequest{
    ....
  }
  
  message LoginResponse{
    ...
  }
  
  消息可以嵌套 
  message SearchResponse{
     message Result{
        string url = 1;
        string title = 2;
     }
  
    string xxx = 1;
    int32  yyy = 2;
    Result ppp = 3;
  }
  
  // message外部使用SearchResponse.Result
  message AAA{
    string xxx = 1;
    SearchResponse.Result yyy = 2;
  }
  
  oneof [其中一个]
  message SimpleMessage{
     oneof test_oneof{
        string name = 1;
        int32  age = 2;
     }
     
     test_oneof xxx
  }
  ~~~

- 服务

  ~~~protobuf
  service HelloService{
     rpc hello(HelloRequest) returns(HelloResponse){}
  }
  // 里面是可以定义多个服务方法。
  // 定义多个服务接口
  // gPRC 服务 4个服务方式 。
  ~~~



# 实战开始

自动编译proto文件称java文件

https://github.com/grpc/grpc-java
