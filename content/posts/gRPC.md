---
title: "GRPC"
date: 2023-07-30T15:53:16+10:00
draft: false
authors: [Sad_man]
tags: [面试]
categories: [rpc]
series: [学习笔记]
series_weight: 1
---

# 一、环境配置

安装

`brew install protobuf`

```
protoc --proto_path=/Users/shengquan/Downloads/CS/Program/Java/Raft_KV_Platform/src/main/proto/ --java_out=/Users/shengquan/Downloads/CS/Program/Java/Raft_KV_Platform/src/main/java/ --grpc-java_out=/Users/shengquan/Downloads/CS/Program/Java/Raft_KV_Platform/src/main/java/ raft.proto

```

# 二、

## gRPC 核心的设计思路 

            1. 

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





————————————————————————————

# 一、核心思路

## 1. 核心思路

~~~markdown
1. gRPC 核心的设计思路 
         1. 网络通信 ---> gRPC自己封装网络通信的部分 提供多种语言的 网络通信的封装 （C Java[Netty] GO)
         2. 协议    ---> HTTP2 传输数据的时候 二进制数据内容。 支持双向流（双工）连接的多路复用。
         3. 序列化   ---> 基本文本 JSON  基于二进制 Java原生序列化方式 Thrift二进制的序列化 压缩二级制序列化。
                         protobuf (Protocol Buffers) google开源一种序列化方式  时间效率和空间效率是JSON的3---5倍。
                         IDL语言 
         4. 代理的创建 --->让调用者像调用本地方法那样 去调用远端的服务方法。
                          stub


2. gRPC的好处 
   1. 高效的进行进程间通信。
   2. 支持多种语言 原生支持 C  Go Java实现。C语言版本上扩展 C++ C# NodeJS Python Ruby PHP..
   3. 支持多平台运行 Linux Android IOS MacOS Windows。
   4. gPRC序列化方式采用protobuf，效率高。
   5. 使用Http2协议
   6. 大厂的背书
~~~

## 2. Http2.0协议

~~~markdown
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
~~~

![image-20230305203847598](/Users/shengquan/my_website/content/posts/gRPC.assets/image-20230305203847598.png)

## 3 Protocol Buffers [protobuf]

~~~markdown
1. protobuf 是一种与编程语言无关【IDL】,与具体的平台无关【OS】。他定义的中间语言，可以方便的在client 于 server中进行RPC的数据传输。
2. protobuf 两种版本 proto2 proto3,但是目前主流应用的都是proto3。
3. protobuf主要安装protobuf的编译器，编译器目的，可以把protobuf的IDL语言，转换成具体某一种开发语言。
~~~

## 4 protobuf编译器的安装

~~~markdown
https://github.com/protocolbuffers/protobuf/releases
windows 版本
  1. 直接解压缩 方式在一个特定的目录下面
  2. 直接配置环境变量 path
     protoc --version
mac    版本
  brew install protobuf 
     protoc --version
~~~

![image-20230305205216711](/Users/shengquan/my_website/content/posts/gRPC.assets/image-20230305205216711.png)

## 5 protobuf IDEA的插件

~~~markdown
1. 2021.2版本后面的新版本 IDEA内置了Protobuf插件
2. 之前版本 可以选装第三方Protobuf插件
3. 二者不能共存。
~~~

## 6. protobuf的语法详解

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
  option java_package = "com.suns";
  
  #指定的protobuf生成的外部类的名字（管理内部类【内部类才是真正开发使用】）
  option java_outer_classname = "UserServce";
  
  ~~~

- 逻辑包【了解】

  ~~~markdown
  # 对于protobuf对于文件内容的管理
  package xxx;
  ~~~

- 导入

  ~~~markdown
  UserService.proto
  
  OrderService.proto
    import "xxx/UserService.proto";
  ~~~

- 基本类型

  ![image-20230307104440839](/Users/shengquan/my_website/content/posts/gRPC.assets/image-20230307104440839.png)

- 枚举

  ~~~protobuf
  enum SEASON{
     SPRING = 0;
     SUMMER = 1;
  
  }
  枚举的值 必须是0开始 
  ~~~

- 消息 Message 

  ~~~protobuf
  message LoginRequest {
     string username = 1;
     singular string password = 2;
     int32  age = 3;
  }
  
  编号 从1开始 到2^29-1，只有29位是因为还有5中wire types需要表示，5中方式需要3位来表示。
  
  注意：19000 - 19999 不能用这个区间内的编号，因为他是protobuf自己保留的。
  
  - singular : 这个字段的值 只能是0个或1个 （默认关键字）  null   "123456"
  - repeated
  
  message Result{
     string content = 1;
     repeated string stutas = 2; //这个字段 返回值 是多个 等价于 Java List Protobuf getStatusList()-->List
  }
  
  protobuf [grpc]
  可以定义多个消息 
  
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
  
  SearchResponse.Result
  
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

  message布局

  一个 Protocol Buffers 的 message 在被序列化后的字节流中的**布局**大致如下：

  - 每个字段都会被编码为一连串的字节。字段的编码以字段的 tag 开始，tag 是字段的编号和字段的 wire type 编码后的结果。tag 的编码也是一个 varint。

  - 字段的 tag 之后是字段的值，其编码方式取决于字段的 wire type。例如，如果 wire type 是 0（Varint），那么字段的值就会被编码为 varint。如果 wire type 是 2（Length-delimited），那么字段的值将先是一个表示长度的 varint，后是指定长度的字节。

  - 字段按照在 message 中声明的顺序出现。但是，由于 Protocol Buffers 支持向前和向后兼容性，所以字段可以在任何顺序出现，甚至可以出现多次（尽管这样会浪费空间）。

  - 重复的字段（使用 "repeated" 关键字声明的）可以被编码为多···个连续的字段（每个字段都有自己的 tag 和值），或者被编码为一个字段（这个字段的 wire type 是 2，也就是 Length-delimited），这个字段的值是所有重复字段的值的串联。

  - 对于嵌套的 message，其编码方式与顶层的 message 相同，都是 tag + value 的形式，只不过 value 的内容是内部的 message。

  这样的编码方式可以确保即使消息定义发生改变，也可以正确地解析消息中的字段。如果字节流中有未知的字段（也就是在消息定义中不存在的字段），那么这些字段可以被安全地忽略或按原样保留，以便在转发时使用。

  

  

  在 Protocol Buffers 中，每一个字段的类型都会被编码为一个 wire type，这用于表示该字段在字节流中的编码方式。Protobuf 定义了 5 种不同的 wire types：

  1. Varint（变长整数，wire type = 0）: 用于编码 bool、int32、int64、uint32、uint64、sint32、sint64、enum 等类型。

  2. 64-bit（64位固定长度，wire type = 1）: 用于编码 double、fixed64、sfixed64 等类型。

  3. Length-delimited（长度有限的，wire type = 2）: 用于编码 string、bytes、embedded messages、packed repeated fields 等类型。

  4. Start group（开始组，已废弃，wire type = 3）: 用于组的开始标记，但是这个类型已经在 Protocol Buffers v2 及以后的版本中废弃了。

  5. End group（结束组，已废弃，wire type = 4）: 用于组的结束标记，但是这个类型也已经在 Protocol Buffers v2 及以后的版本中废弃了。

  6. 32-bit（32位固定长度，wire type = 5）: 用于编码 float、fixed32、sfixed32 等类型。

  所以，wire type 不仅表示字段在字节流中的编码方式，还决定了该字段在序列化和反序列化过程中的处理方式。

- 服务

  ~~~protobuf
  service HelloService{
     rpc hello(HelloRequest) returns(HelloResponse){}
  }
  # 里面是可以定义多个服务方法。
  # 定义多个服务接口
  # gPRC 服务 4个服务方式 。
  ~~~

# 二、第一个gPRC的开发

项目结构

~~~markdown
1. xxxx-api 模块 
   定义 protobuf idl语言 
   并且通过命令创建对应的代码
   2. service 
2. xxxx-server模块
   1. 实现api模块中定义的服务接口
   2. 发布gRPC服务 (创建服务端程序)
3. xxxx-clien模块
   1. 创建服务端stub(代理)
   2. 基于代理（stub) RPC调用。
~~~

api模块

~~~markdown
1. .proto文件 书写protobuf的IDL
2. [了解]protoc命令 把proto文件中的IDL 转换成编程语言 
   protoc --java_out=/xxx/xxx  /xxx/xxx/xx.proto
3. maven插件 进行protobuf IDL文件的编译，并把他放置IDEA具体位置。
~~~

~~~xml
pom.xml
 <dependencies>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-netty-shaded</artifactId>
            <version>1.52.1</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-protobuf</artifactId>
            <version>1.52.1</version>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-stub</artifactId>
            <version>1.52.1</version>
        </dependency>
        <dependency> <!-- necessary for Java 9+ -->
            <groupId>org.apache.tomcat</groupId>
            <artifactId>annotations-api</artifactId>
            <version>6.0.53</version>
            <scope>provided</scope>
        </dependency>
 </dependencies>

<build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.7.1</version>
            </extension>
        </extensions>
        <plugins>
            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.6.1</version>
                <configuration>
                    <protocArtifact>com.google.protobuf:protoc:3.21.7:exe:${os.detected.classifier}</protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.52.1:exe:${os.detected.classifier}</pluginArtifact>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

~~~

![image-20230308132952661](/Users/shengquan/my_website/content/posts/gRPC.assets/image-20230308132952661.png)

- xxxx-server 服务端模块的开发

  ~~~java
  1. 实现业务接口 添加具体的功能 （MyBatis+MySQL)
     public class HelloServiceImpl extends HelloServiceGrpc.HelloServiceImplBase {
      /*
        1. 接受client提交的参数  request.getParameter()
        2. 业务处理 service+dao 调用对应的业务功能。
        3. 提供返回值
       */
      @Override
      public void hello(HelloProto.HelloRequest request, StreamObserver<HelloProto.HelloResponse> responseObserver) {
          //1.接受client的请求参数
          String name = request.getName();
          //2.业务处理
          System.out.println("name parameter "+name);
          //3.封装响应
          //3.1 创建相应对象的构造者
          HelloProto.HelloResponse.Builder builder = HelloProto.HelloResponse.newBuilder();
          //3.2 填充数据
          builder.setResult("hello method invoke ok");
          //3.3 封装响应
          HelloProto.HelloResponse helloResponse = builder.build();
  
          responseObserver.onNext(helloResponse);
          responseObserver.onCompleted();;
      }
  }
  2. 创建服务端 （Netty)
  public class GprcServer1 {
      public static void main(String[] args) throws IOException, InterruptedException {
          //1. 绑定端口 
          ServerBuilder serverBuilder = ServerBuilder.forPort(9000);
          //2. 发布服务
          serverBuilder.addService(new HelloServiceImpl());
          //serverBuilder.addService(new UserServiceImpl());
          //3. 创建服务对象
          Server server = serverBuilder.build();
          
          server.start();
          server.awaitTermination();;
      }
  }
  ~~~

- xxx-client 模块

  ~~~java
  1. client通过代理对象完成远端对象的调用
  
  public class GprcClient1 {
      public static void main(String[] args) {
          //1 创建通信的管道
          ManagedChannel managedChannel = ManagedChannelBuilder.forAddress("localhost", 9000).usePlaintext().build();
          //2 获得代理对象 stub
          try {
              HelloServiceGrpc.HelloServiceBlockingStub helloService = HelloServiceGrpc.newBlockingStub(managedChannel);
              //3. 完成RPC调用
              //3.1 准备参数
              HelloProto.HelloRequest.Builder builder = HelloProto.HelloRequest.newBuilder();
              builder.setName("sunshuai");
              HelloProto.HelloRequest helloRequest = builder.build();
              //3.1 进行功能rpc调用，获取相应的内容
              HelloProto.HelloResponse helloResponse = helloService.hello(helloRequest);
              String result = helloResponse.getResult();
              System.out.println("result = " + result);
          } catch (Exception e) {
              throw new RuntimeException(e);
          }finally {
              managedChannel.shutdown();
          }
      }
  }
  ~~~

- 注意事项

  ~~~markdown
  服务端 处理返回值时
  responseObserver.onNext(helloResponse1);  //通过这个方法 把响应的消息 回传client
  responseObserver.onCompleted();           //通知client 整个服务结束。底层返回标记 
                                            // client就会监听标记 【grpc做的】
                                            
  requestObserver.onNext(helloRequest1);
  requestObserver.onCompleted();
  ~~~

###### 4.3.5gRpc的四种通信方式

- 四种通信方式

  ~~~markdown
  1. 简单rpc 一元rpc (Unary RPC)
  2. 服务端流式RPC   (Server Streaming RPC)
  3. 客户端流式RPC   (Client Streaming RPC)
  4. 双向流RPC (Bi-directional Stream RPC)
  ~~~

- 简单RPC(一元RPC)

  ~~~markdown
  1. 第一个RPC程序，实际上就是一元RPC
  ~~~

  - 特点

    ~~~markdown
    当client发起调用后，提交数据，并且等待 服务端响应。
    开发过程中，主要采用就是一元RPC的这种通信方式
    ~~~

    ![image-20230309153848531](/Users/shengquan/my_website/content/posts/gRPC.assets/image-20230309153848531.png)

  - 语法

    ~~~protobuf
    service HelloService{
      rpc hello(HelloRequest) returns (HelloResponse){}
      rpc hello1(HelloRequest1) returns (HelloResponse1){}
    }
    ~~~

- 服务端流式RPC 

  ~~~markdown
  一个请求对象，服务端可以回传多个结果对象。
  ~~~

  - 特点

    ![image-20230309154817840](./RPC-Day1/image-20230309154817840.png)

  - 使用场景

    ~~~markdown
    client  --------> Server
            股票标号
            <-------
             某一个时刻的 股票的行情
    ~~~

  - 语法

    ~~~markdown
    service HelloService{
      rpc hello(HelloRequest) returns (stream HelloResponse){}
      rpc hello1(HelloRequest1) returns (HelloResponse1){}
    }
    ~~~

  - 关键代码

    ~~~java
    服务端
    public void c2ss(HelloProto.HelloRequest request, StreamObserver<HelloProto.HelloResponse> responseObserver) {
            //1 接受client的请求参数
            String name = request.getName();
            //2 做业务处理
            System.out.println("name = " + name);
            //3 根据业务处理的结果，提供响应
            for (int i = 0; i < 9; i++) {
                HelloProto.HelloResponse.Builder builder = HelloProto.HelloResponse.newBuilder();
                builder.setResult("处理的结果 " + i);
                HelloProto.HelloResponse helloResponse = builder.build();
    
                responseObserver.onNext(helloResponse);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
            responseObserver.onCompleted();
        }
    客户端
    public class GprcClient3 {
        public static void main(String[] args) {
            ManagedChannel managedChannel = ManagedChannelBuilder.forAddress("localhost", 9000).usePlaintext().build();
            try {
                HelloServiceGrpc.HelloServiceBlockingStub helloService = HelloServiceGrpc.newBlockingStub(managedChannel);
    
                HelloProto.HelloRequest.Builder builder = HelloProto.HelloRequest.newBuilder();
                builder.setName("sunshuai");
                HelloProto.HelloRequest helloRequest = builder.build();
                Iterator<HelloProto.HelloResponse> helloResponseIterator = helloService.c2ss(helloRequest);
                while (helloResponseIterator.hasNext()) {
                    HelloProto.HelloResponse helloResponse = helloResponseIterator.next();
                    System.out.println("helloResponse.getResult() = " + helloResponse.getResult());
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
            finally {
                managedChannel.shutdown();
            }
        }
    }
    
    监听 异步方式 处理服务端流式RPC的开发
    1. api
    2. 服务端 
    3. 客户端 
       public class GrpcClient4 {
        public static void main(String[] args) {
            ManagedChannel managedChannel = ManagedChannelBuilder.forAddress("localhost", 9000).usePlaintext().build();
    
            try {
    
                HelloServiceGrpc.HelloServiceStub helloService = HelloServiceGrpc.newStub(managedChannel);
    
                HelloProto.HelloRequest.Builder builder = HelloProto.HelloRequest.newBuilder();
                builder.setName("xiaohei");
                HelloProto.HelloRequest helloRequest = builder.build();
    
                helloService.c2ss(helloRequest, new StreamObserver<HelloProto.HelloResponse>() {
                    @Override
                    public void onNext(HelloProto.HelloResponse value) {
                        //服务端 响应了 一个消息后，需要立即处理的话。把代码写在这个方法中。
                        System.out.println("服务端每一次响应的信息 " + value.getResult());
                    }
    
                    @Override
                    public void onError(Throwable t) {
    
                    }
    
                    @Override
                    public void onCompleted() {
                        //需要把服务端 响应的所有数据 拿到后，在进行业务处理。
                        System.out.println("服务端响应结束 后续可以根据需要 在这里统一处理服务端响应的所有内容");
                    }
                });
    
                managedChannel.awaitTermination(12, TimeUnit.SECONDS);
    
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                managedChannel.shutdown();
            }
        }
    }
    ~~~

- 客户端流式RPC

  ~~~markdown
  客户端发送多个请求对象，服务端只返回一个结果
  ~~~

  ![image-20230310142153479](./RPC-Day1/image-20230310142153479.png)

  - 应用场景

    ~~~markdown
    IOT(物联网 【传感器】) 向服务端 发送数据
    ~~~

  - proto

    ~~~protobuf
    rpc cs2s(stream HelloRequest) returns (HelloResponse){}
    ~~~

  - 开发

    ~~~java
    1. api
       rpc cs2s(stream HelloRequest) returns (HelloResponse){}
    2. 服务端开发
       public StreamObserver<HelloProto.HelloRequest> cs2s(StreamObserver<HelloProto.HelloResponse> responseObserver) {
            return new StreamObserver<HelloProto.HelloRequest>() {
                @Override
                public void onNext(HelloProto.HelloRequest value) {
                    System.out.println("接受到了client发送一条消息 " + value.getName());
                }
    
                @Override
                public void onError(Throwable t) {
    
                }
    
                @Override
                public void onCompleted() {
                    System.out.println("client的所有消息 都发送到了 服务端 ....");
    
                    //提供响应：响应的目的：当接受了全部client提交的信息，并处理后，提供相应
                    HelloProto.HelloResponse.Builder builder = HelloProto.HelloResponse.newBuilder();
                    builder.setResult("this is result");
                    HelloProto.HelloResponse helloResponse = builder.build();
    
                    responseObserver.onNext(helloResponse);
                    responseObserver.onCompleted();
                }
            };
        }
    3. 客户端开发
       public class GrpcClient5 {
        public static void main(String[] args) {
            ManagedChannel managedChannel = ManagedChannelBuilder.forAddress("localhost", 9000).usePlaintext().build();
            try {
                HelloServiceGrpc.HelloServiceStub helloService = HelloServiceGrpc.newStub(managedChannel);
    
                StreamObserver<HelloProto.HelloRequest> helloRequestStreamObserver = helloService.cs2s(new StreamObserver<HelloProto.HelloResponse>() {
                    @Override
                    public void onNext(HelloProto.HelloResponse value) {
                        // 监控响应
                        System.out.println("服务端 响应 数据内容为 " + value.getResult());
    
                    }
    
                    @Override
                    public void onError(Throwable t) {
    
                    }
    
                    @Override
                    public void onCompleted() {
                        System.out.println("服务端响应结束 ... ");
                    }
                });
    
                //客户端 发送数据 到服务端  多条数据 ，不定时...
                for (int i = 0; i < 10; i++) {
                    HelloProto.HelloRequest.Builder builder = HelloProto.HelloRequest.newBuilder();
                    builder.setName("sunshuai " + i);
                    HelloProto.HelloRequest helloRequest = builder.build();
    
                    helloRequestStreamObserver.onNext(helloRequest);
                    Thread.sleep(1000);
                }
    
                helloRequestStreamObserver.onCompleted();
    
                managedChannel.awaitTermination(12, TimeUnit.SECONDS);
    
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                managedChannel.shutdown();
            }
        }
    }
    ~~~

- 双向流式RPC

  ~~~markdown
  客户端可以发送多个请求消息，服务端响应多个响应消息
  ~~~

  ![image-20230310151339486](/Users/shengquan/my_website/content/posts/gRPC.assets/image-20230310151339486.png)

  - 应用场景

    ~~~markdown
    聊天室
    ~~~

  - 编码

    ~~~java
    1. api
        rpc cs2ss(stream HelloRequest) returns (stream HelloResponse){}
    2. 服务端
        public StreamObserver<HelloProto.HelloRequest> cs2ss(StreamObserver<HelloProto.HelloResponse> responseObserver) {
             return new StreamObserver<HelloProto.HelloRequest>() {
                 @Override
                 public void onNext(HelloProto.HelloRequest value) {
                     System.out.println("接受到client 提交的消息 "+value.getName());
                     responseObserver.onNext(HelloProto.HelloResponse.newBuilder().setResult("response "+value.getName()+" result ").build());
                 }
    
                 @Override
                 public void onError(Throwable t) {
    
                 }
    
                 @Override
                 public void onCompleted() {
                     System.out.println("接受到了所有的请求消息 ... ");
                     responseObserver.onCompleted();
                 }
             };
        }
    3. 客户端
      public class GrpcClient6 {
        public static void main(String[] args) {
            ManagedChannel managedChannel = ManagedChannelBuilder.forAddress("localhost", 9000).usePlaintext().build();
            try {
                HelloServiceGrpc.HelloServiceStub helloService = HelloServiceGrpc.newStub(managedChannel);
    
                StreamObserver<HelloProto.HelloRequest> helloRequestStreamObserver = helloService.cs2ss(new StreamObserver<HelloProto.HelloResponse>() {
                    @Override
                    public void onNext(HelloProto.HelloResponse value) {
                        System.out.println("响应的结果 "+value.getResult());
                    }
    
                    @Override
                    public void onError(Throwable t) {
    
                    }
    
                    @Override
                    public void onCompleted() {
                        System.out.println("响应全部结束...");
                    }
                });
    
    
                for (int i = 0; i < 10; i++) {
                    helloRequestStreamObserver.onNext(HelloProto.HelloRequest.newBuilder().setName("sunshuai " + i).build());
                }
                helloRequestStreamObserver.onCompleted();
    
                managedChannel.awaitTermination(12, TimeUnit.SECONDS);
    
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                managedChannel.shutdown();
            }
        }
    }
    ~~~

###### 4.3.6 gPRC代理方式

~~~markdown
1. BlockingStub
   阻塞 通信方式 
2. Stub
   异步 通过监听处理的
3. FutureStub
   同步 异步 NettyFuture
   1. FutureStub只能应用 一元RPC 
~~~

~~~java
public class GrpcClient7 {
    public static void main(String[] args) {
        ManagedChannel managedChannel = ManagedChannelBuilder.forAddress("localhost", 9000).usePlaintext().build();
        try {
            TestServiceGrpc.TestServiceFutureStub testServiceFutureStub = TestServiceGrpc.newFutureStub(managedChannel);
            ListenableFuture<TestProto.TestResponse> responseListenableFuture = testServiceFutureStub.testSuns(TestProto.TestRequest.newBuilder().setName("xiaojren").build());


            /* 同步操作
            TestProto.TestResponse testResponse = responseListenableFuture.get();
            System.out.println(testResponse.getResult());*/

          /*  responseListenableFuture.addListener(() -> {
                System.out.println("异步的rpc响应 回来了....");
            }, Executors.newCachedThreadPool());*/

            Futures.addCallback(responseListenableFuture, new FutureCallback<TestProto.TestResponse>() {
                @Override
                public void onSuccess(TestProto.TestResponse result) {
                    System.out.println("result.getResult() = " + result.getResult());
                }

                @Override
                public void onFailure(Throwable t) {

                }
            }, Executors.newCachedThreadPool());


            System.out.println("后续的操作....");

            managedChannel.awaitTermination(12, TimeUnit.SECONDS);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            managedChannel.shutdown();
        }
    }
}
~~~

#### 5. gPRC与SpringBoot整合

##### 5.1 gRPC和SpringBoot整合的思想 

~~~markdown
1. grpc-server
2. grpc-client 
~~~

###### SpringBoot与GRPC整合的过程中 对于服务端做了什么封装

![image-20230312191718736](/Users/shengquan/my_website/content/posts/gRPC.assets/image-20230312191718736.png)

- 搭建开发环境

  ~~~markdown
  1. 搭建SpringBoot的开发环境
     
  2. 引入与Grpc相关的内容
      <dependency>
            <groupId>com.suns</groupId>
            <artifactId>rpc-grpc-api</artifactId>
            <version>1.0-SNAPSHOT</version>
       </dependency>
  
      <dependency>
          <groupId>net.devh</groupId>
          <artifactId>grpc-server-spring-boot-starter</artifactId>
          <version>2.14.0.RELEASE</version>
      </dependency>
  
  ~~~

- 开发服务

  ~~~java
  // 重复 多次 
  @GrpcService
  public class HelloServiceImpl extends HelloServiceGrpc.HelloServiceImplBase {
      @Override
      public void hello(HelloProto.HelloRequest request, StreamObserver<HelloProto.HelloResponse> responseObserver) {
          String name = request.getName();
          System.out.println("name is " + name);
  
          responseObserver.onNext(HelloProto.HelloResponse.newBuilder().setResult("this is result").build());
          responseObserver.onCompleted();
      }
  }
  // application.yml
  # 核心配置的 就是gRPC服务的端口号
  spring:
    application:
      name: boot-server
  
    main:
      web-application-type: none
  
  grpc:
    server:
      port: 9000
  
  ~~~

- 客户端

  环境搭建

  ~~~markdown
   <dependency>
      <groupId>net.devh</groupId>
      <artifactId>grpc-client-spring-boot-starter</artifactId>
      <version>2.14.0.RELEASE</version>
   </dependency>
  ~~~

  编码

  ~~~markdown
  1. yml
  grpc:
    client:
      grpc-server:
        address: 'static://127.0.0.1:9000'
        negotiation-type: plaintext
        
  2. 注入stub
  
  @GrpcClient("grpc-server")
  private HelloServiceGrpc.HelloServiceBlockingStub stub;
  ~~~

#### 6. gPRC的高级应用

~~~markdown
1. 拦截器  一元拦截器 
2. Stream Tracer [监听流]  流拦截器 
3. Retry Policy 客户端重试 
4. NameResolver 
5. 负载均衡 （pick-first , 轮训）
6. grpc与微服务的整合 
   序列化 （protobuf) Dobbo
   grpc              Dobbo
   grpc              GateWay
   grpc              JWT
   grpc              Nacos2.0
   grpc              OpenFeign 
~~~

##### 6.1 拦截器

~~~markdown
1. 拦截器的作用
   鉴权 ，数据校验 ，限流...
2. 常见的拦截器 
   web        Filter
   springmvc  Interceptor 
   SS         Filter
   Netty      handler
3. gRPC的拦截器 
   1.  一元请求的 拦截器 
       客户端 【请求 响应】
       服务端 【请求 响应】
   2.  流式请求的 拦截器  （Stream Tracer)
       客户端 【请求 响应】
       服务端 【请求 响应】
~~~

###### 6.1.1 一元拦截器

![image-20230316112142470](/Users/shengquan/my_website/content/posts/gRPC.assets/image-20230316112142470.png)

- 简单客户端拦截器的开发

  ~~~java
  //1 开发拦截器 
  @Slf4j
  public class CustomClientInterceptor implements ClientInterceptor {
      @Override
      public <ReqT, RespT> ClientCall<ReqT, RespT> interceptCall(MethodDescriptor<ReqT, RespT> method, CallOptions callOptions, Channel next) {
          log.debug("这是一个拦截启动的处理 ,统一的做了一些操作 ....");
          return next.newCall(method, callOptions);
      }
  }
  
  
  //2 在客户端进行设置 
   ManagedChannel managedChannel = ManagedChannelBuilder.forAddress("localhost", 9000)
                  .usePlaintext()
                  .intercept(new CustomClientInterceptor())
                  .build();
  细节：
    usePlaintext()
    intercept(）
    调用的先后顺序没有必须关联，谁先谁后都行。
  ~~~

- 简单客户端拦截器问题

  ~~~markdown
  1. 只能拦截请求，不能拦截响应。
  2. 即使拦截了请求操作，粒度业务过于宽泛，不精准。
     1. 开始阶段 2. 设置消息数量 3.发送数据阶段 4.半连接阶段。
  ~~~

- 复杂客户端拦截器 

  ~~~java
  1. 拦截请求，拦截响应。
  2. 分阶段的进行 请求 响应的拦截
  
  渗透的 ClientCall[方法]  装饰器设计模式 典型的使用场景。
  
  public class CustomClientInterceptor implements ClientInterceptor {
      @Override
      public <ReqT, RespT> ClientCall<ReqT, RespT> interceptCall(MethodDescriptor<ReqT, RespT> method, CallOptions callOptions, Channel next) {
          log.debug("这是一个拦截启动的处理 ,统一的做了一些操作 ....");
          /*
             如果我们需要用复杂客户端拦截器 ，就需要对原始的ClientCall进行包装
             那么这个时候，就不能反悔原始ClientCall对象，
             应该返回 包装的ClientCall ---> CustomForwardingClientClass
           */
          //return next.newCall(method, callOptions);
          return new CustomForwardingClientClass<>(next.newCall(method, callOptions));
      }
  }
  
  /*
     这个类型 适用于控制 拦截 请求发送各个环节
   */
  @Slf4j
  class CustomForwardingClientClass<ReqT, RespT> extends ClientInterceptors.CheckedForwardingClientCall<ReqT, RespT> {
  
      protected CustomForwardingClientClass(ClientCall<ReqT, RespT> delegate) {
          super(delegate);
      }
  
      @Override
      //开始调用
      // 目的 看一个这个RPC请求是不是可以被发起。
      protected void checkedStart(Listener<RespT> responseListener, Metadata headers) throws Exception {
          log.debug("发送请求数据之前的检查.....");
          //真正的去发起grpc的请求
          // 是否真正发送grpc的请求，取决这个start方法的调用
          //delegate().start(responseListener, headers);
          delegate().start(new CustomCallListener<>(responseListener), headers);
      }
  
      @Override
      //指定发送消息的数量
      public void request(int numMessages) {
          //添加一些功能
          log.debug("request 方法被调用 ....");
          super.request(numMessages);
      }
  
      @Override
      //发送消息 缓冲区
      public void sendMessage(ReqT message) {
          log.debug("sendMessage 方法被调用... {} ", message);
          super.sendMessage(message);
      }
  
      @Override
      //开启半连接 请求消息无法发送，但是可以接受响应的消息
      public void halfClose() {
          log.debug("halfClose 方法被调用... 开启了半连接");
          super.halfClose();
      }
  }
  
  /*
     用于监听响应，并对响应进行拦截
   */
  @Slf4j
  class CustomCallListener<RespT> extends ForwardingClientCallListener.SimpleForwardingClientCallListener<RespT> {
      protected CustomCallListener(ClientCall.Listener<RespT> delegate) {
          super(delegate);
      }
  
      @Override
      public void onHeaders(Metadata headers) {
          log.info("响应头信息 回来了......");
          super.onHeaders(headers);
      }
  
      @Override
      public void onMessage(RespT message) {
          log.info("响应的数据 回来了.....{} ", message);
          super.onMessage(message);
      }
  }
  
  
  onHeaders这个方法 会在onNext调用后 监听到数据
  onMessage这个方法 会在onCompleted调用后 监听到数据
  ~~~

- 简单服务端拦截器

  ~~~java
  @Slf4j
  public class CustomServerInterceptor implements ServerInterceptor {
      @Override
      public <ReqT, RespT> ServerCall.Listener<ReqT> interceptCall(ServerCall<ReqT, RespT> call, Metadata headers, ServerCallHandler<ReqT, RespT> next) {
          //在服务器端 拦截请求操作的功能 写在这个方法中
          log.debug("服务器端拦截器生效.....");
          return next.startCall(call, headers);
      }
  }
  
  
  public class GrpcServer {
      public static void main(String[] args) throws InterruptedException, IOException {
          ServerBuilder<?> serverBuilder = ServerBuilder.forPort(9000);
          serverBuilder.addService(new HelloServiceImpl());
          serverBuilder.intercept(new CustomServerInterceptor());
          Server server = serverBuilder.build();
  
          server.start();
          server.awaitTermination();
      }
  }
  ~~~

- 简单服务端拦截器问题

  ~~~markdown
  1. 拦截请求发送过来的数据，无法处理响应的数据
  2. 拦截力度过于宽泛
  ~~~

- 复杂服务端拦截器

  ~~~markdown
  1. 拦截请求的数据，拦截响应的数据
  2. 粒度细致
  ~~~

  - 请求数据的拦截

    ~~~java
    @Slf4j
    public class CustomServerInterceptor implements ServerInterceptor {
        @Override
        public <ReqT, RespT> ServerCall.Listener<ReqT> interceptCall(ServerCall<ReqT, RespT> call, Metadata headers, ServerCallHandler<ReqT, RespT> next) {
            //在服务器端 拦截请求操作的功能 写在这个方法中
            log.debug("服务器端拦截器生效.....");
            //默认返回的ServerCall.Listener仅仅能够完成请求数据的监听，单没有拦截功能
            //所以要做扩展，采用包装器设计模式。
            //return next.startCall(call, headers);
            return new CustomServerCallListener<>(next.startCall(call, headers));
        }
    }
    
    @Slf4j
    class CustomServerCallListener<ReqT> extends ForwardingServerCallListener.SimpleForwardingServerCallListener<ReqT> {
        protected CustomServerCallListener(ServerCall.Listener<ReqT> delegate) {
            super(delegate);
        }
    
        @Override
        //准备接受请求数据
        public void onReady() {
            log.debug("onRead Method Invoke....");
            super.onReady();
        }
    
        @Override
        public void onMessage(ReqT message) {
            log.debug("接受到了 请求提交的数据  {} ", message);
            super.onMessage(message);
        }
    
        @Override
        public void onHalfClose() {
            log.debug("监听到了 半连接...");
            super.onHalfClose();
        }
    
        @Override
        public void onComplete() {
            log.debug("服务端 onCompleted()...");
            super.onComplete();
        }
    
        @Override
        public void onCancel() {
            log.debug("出现异常后 会调用这个方法... 关闭资源的操作");
            super.onCancel();
        }
    }
    
    ~~~

  - 拦截响应

    ~~~java
    目的：通过自定义的ServerCall 包装原始的ServerCall 增加对于响应拦截的功能
    @Slf4j
    class CustomServerCall<ReqT, RespT> extends ForwardingServerCall.SimpleForwardingServerCall<ReqT, RespT> {
    
        protected CustomServerCall(ServerCall<ReqT, RespT> delegate) {
            super(delegate);
        }
    
        @Override
        //指定发送消息的数量 【响应消息】
        public void request(int numMessages) {
            log.debug("response 指定消息的数量 【request】");
            super.request(numMessages);
        }
    
        @Override
        //设置响应头
        public void sendHeaders(Metadata headers) {
            log.debug("response 设置响应头 【sendHeaders】");
            super.sendHeaders(headers);
        }
    
        @Override
        //响应数据
        public void sendMessage(RespT message) {
            log.debug("response 响应数据  【send Message 】 {} ", message);
            super.sendMessage(message);
        }
    
        @Override
        //关闭连接
        public void close(Status status, Metadata trailers) {
            log.debug("respnse 关闭连接 【close】");
            super.close(status, trailers);
        }
    }
    
    目的：就是把自定义的ServerCall与gRpc服务端进行整合
    @Slf4j
    public class CustomServerInterceptor implements ServerInterceptor {
        @Override
        public <ReqT, RespT> ServerCall.Listener<ReqT> interceptCall(ServerCall<ReqT, RespT> call, Metadata headers, ServerCallHandler<ReqT, RespT> next) {
            //在服务器端 拦截请求操作的功能 写在这个方法中
            log.debug("服务器端拦截器生效.....");
          
            //1. 包装ServerCall 处理服务端响应拦截
            CustomServerCall<ReqT,RespT> reqTRespTCustomServerCall = new CustomServerCall<>(call);
            //2. 包装Listener   处理服务端请求拦截
            CustomServerCallListener<ReqT> reqTCustomServerCallListener = new CustomServerCallListener<>(next.startCall(reqTRespTCustomServerCall, headers));
            return reqTCustomServerCallListener;
        }
    }
    ~~~

###### 6.1.2 流式拦截器

~~~markdown
主要应用 gRPC除了 一元RPC之外的其他形式RPC操作的拦截工作
1. 服务端流式RPC
2. 客户端流式RPC
3. 双向流式RPC
~~~

- 一元通信方式 和 流式通信方式的区别

  ~~~markdown
  对比 一元通信方式，流式通信方式的特点是消息多，而且不是一次性通信，是分批分期。这个时候一元拦截器就无法细粒度的拦截特定的消息。
  所以需要流式拦截器 解决这个问题。流式拦截器【Stream Tracer】
  ~~~

- 客户端流式拦截器的开发

  ~~~markdown
  1. 拦截请求  拦截响应
  2. 开发思路 
     ClientStreamTracer [拦截请求 拦截响应]
     ClientStreamTracerFactory 
        目的 就是用于创建ClientStreamTracer.
  ~~~

  ~~~java
  @Slf4j
  public class CustomerClientInterceptor implements ClientInterceptor {
      @Override
      public <ReqT, RespT> ClientCall<ReqT, RespT> interceptCall(MethodDescriptor<ReqT, RespT> method, CallOptions callOptions, Channel next) {
          log.debug("执行客户端拦截器...");
  
  
          //把自己开发的ClientStreamTracerFactory融入到gRPC体系
          callOptions = callOptions.withStreamTracerFactory(new CustomClientStreamTracerFactory<>());
          return next.newCall(method, callOptions);
      }
  }
  
  class CustomClientStreamTracerFactory<ReqT, RespT> extends ClientStreamTracer.Factory {
      @Override
      public ClientStreamTracer newClientStreamTracer(ClientStreamTracer.StreamInfo info, Metadata headers) {
          return new CustomClientStreamTracer<>();
      }
  }
  
  
  /*
   作用： 用于客户端流式拦截：拦截请求 拦截响应
   */
  @Slf4j
  class CustomClientStreamTracer<ReqT, RespT> extends ClientStreamTracer {
      //outbound 对于请求相关操作的拦截
  
      @Override
      //用于输出响应头
      public void outboundHeaders() {
          log.debug("client: 用于输出请求头.....");
          super.outboundHeaders();
      }
  
      @Override
      //设置消息编号
      public void outboundMessage(int seqNo) {
          log.debug("client: 设置流消息的编号 {} ", seqNo);
          super.outboundMessage(seqNo);
      }
  
      @Override
      public void outboundUncompressedSize(long bytes) {
          log.debug("client: 获得未压缩消息的大小 {} ", bytes);
          super.outboundUncompressedSize(bytes);
      }
  
      @Override
      //用于获得 输出消息的大小
      public void outboundWireSize(long bytes) {
          log.debug("client: 用于获得 输出消息的大小 {} ", bytes);
          super.outboundWireSize(bytes);
      }
  
      @Override
      //拦截消息发送
      public void outboundMessageSent(int seqNo, long optionalWireSize, long optionalUncompressedSize) {
          log.debug("client: 监控请求操作 outboundMessageSent {} ", seqNo);
          super.outboundMessageSent(seqNo, optionalWireSize, optionalUncompressedSize);
      }
  
  
      //inbound  对于相应相关操作的拦截
      @Override
      public void inboundHeaders() {
          log.debug("用于获得响应头....");
          super.inboundHeaders();
      }
  
      @Override
      public void inboundMessage(int seqNo) {
          log.debug("获得响应消息的编号...{} ",seqNo);
          super.inboundMessage(seqNo);
      }
  
      @Override
      public void inboundWireSize(long bytes) {
          log.debug("获得响应消息的大小...{} ",bytes);
          super.inboundWireSize(bytes);
      }
  
      @Override
      public void inboundMessageRead(int seqNo, long optionalWireSize, long optionalUncompressedSize) {
          log.debug("集中获得消息的编号 ，大小 ，未压缩大小 {} {} {}", seqNo, optionalWireSize, optionalUncompressedSize);
          super.inboundMessageRead(seqNo, optionalWireSize, optionalUncompressedSize);
      }
  
      @Override
      public void inboundUncompressedSize(long bytes) {
          log.debug("获得响应消息未压缩大小 {} ",bytes);
          super.inboundUncompressedSize(bytes);
      }
  
      @Override
      public void inboundTrailers(Metadata trailers) {
          log.debug("响应结束..");
          super.inboundTrailers(trailers);
      }
  }
  ~~~

- 服务端流式拦截器的开发

  ~~~markdown
  1. ServerStreamTracer 拦截
     inbound  请求的拦截
     outbound 响应的拦截 
  2. ServerStreamTracerFactory进行创建
     
  ~~~

  ~~~java
  public class CustomServerStreamFactory extends ServerStreamTracer.Factory {
      @Override
      public ServerStreamTracer newServerStreamTracer(String fullMethodName, Metadata headers) {
          return new CustomServerStreamTracer();
      }
  }
  
  @Slf4j
  class CustomServerStreamTracer extends ServerStreamTracer {
      //inbound 拦截请求
      @Override
      public void inboundMessage(int seqNo) {
          super.inboundMessage(seqNo);
      }
  
      @Override
      public void inboundWireSize(long bytes) {
          super.inboundWireSize(bytes);
      }
  
      @Override
      public void inboundMessageRead(int seqNo, long optionalWireSize, long optionalUncompressedSize) {
          log.debug("server: 获得client发送的请求消息 ...{} {} {}", seqNo, optionalWireSize, optionalUncompressedSize);
          super.inboundMessageRead(seqNo, optionalWireSize, optionalUncompressedSize);
      }
  
      @Override
      public void inboundUncompressedSize(long bytes) {
          super.inboundUncompressedSize(bytes);
      }
  
      //outbound 拦截请求
  
      @Override
      public void outboundMessage(int seqNo) {
          super.outboundMessage(seqNo);
      }
  
  
      @Override
      public void outboundMessageSent(int seqNo, long optionalWireSize, long optionalUncompressedSize) {
          log.debug("server: 响应数据的拦截 ...{} {} {}", seqNo, optionalWireSize, optionalUncompressedSize);
          super.outboundMessageSent(seqNo, optionalWireSize, optionalUncompressedSize);
      }
  
      @Override
      public void outboundWireSize(long bytes) {
          super.outboundWireSize(bytes);
      }
  
      @Override
      public void outboundUncompressedSize(long bytes) {
          super.outboundUncompressedSize(bytes);
      }
  }
  
  serverBuilder.addStreamTracerFactory(new CustomServerStreamFactory());
  ~~~


##### 6.2 客户端重试

~~~markdown
客户端重试保证了系统可靠性。
在RGPC调用过程中看，因为网络延迟 或者抖动，可能操作客户端暂时链接不上服务端的情况，如果出现了此类情况的话。那么可以让client重试连接。尽可能的避免，因为网络问题，影响通信。从而保证了系统的可靠性。
~~~

- 开发

  ~~~java
  1. 服务端 
     模拟网络问题 
  2. 客户端
     {
    "methodConfig": [
      {
        "name": [
          {
            "service": "com.suns.HelloService", //服务的名字 包名 package
            "method": "hello"  //服务方法名字 
          }
        ],
        "retryPolicy": {
          "maxAttempts": 3,         //重试的次数 
          "initialBackoff": "0.5s", //初始重试的延迟时间
          "maxBackof": "30s",       //最大重试的延迟时间
          "backoffMultiplier": 2,   //退避指数 每一次重试的时间间隔，不是不固定，2 4 6...
          "retryableStatusCodes": [ //只有当接受到了这个数组中描述的异常，才会发起重试
            "UNAVAILABLE"
          ]
        }
      }
    ]
  }
     ManagedChannel managedChannel = ManagedChannelBuilder.forAddress("localhost", 9000)
                  .usePlaintext()
                  .defaultServiceConfig(serviceConfig) //设置重试的参数 
                  .enableRetry()  //打开重试 
                  .build();
  ~~~

##### 6.3 命名解析 NameResovler

~~~markdown
1. 什么是命名解析
   本质上就是我们在SOA或者微服务中，所说的注册中心。
2. 为什么需要注册中心(名字解析）？
   rpc集群环境下，通过注册中心（名字解析）统一管理rpc集群 【负载均衡，监控检查....】
3. 目前开发中 常见注册中心 
   Dubbo         zookeeper 
   SpringCloud   eureka nacos consul etcd ...  
4. gRPC 名字解析 注册中心 
   1. 默认gPRC是通过DNS 做注册中心。
   2. 可以用户自定义注册中心 （扩展）
5. 如何进行gRPC注册中心的扩展 【重点】
              zk     etcd   consul 
   开发语言    java    go     go
   算法       Paxos   Raft   Raft
   多数据中心  不支持   不支持  支持 
   web管理    支持     不支持  支持
   ....
~~~

![image-20230319203204763](/Users/shengquan/my_website/content/posts/gRPC.assets/image-20230319203204763.png)

###### 6.3.1 Consul 

~~~markdown
1. Consul是什么？
   HashiCorp公司开源一个工具，用于实现分布式系统中服务发现（注册）与配置。nacos
   https://www.consul.io/
   一站式的解决方案，内置注册中心，健康检查，多数据中心，KeyValue存储。。。
2. Consul作用：
   1. 注册中心
   2. 配置中心 
3. 使用Consul的好处 
   1. 安装简单 ---》自带web管理界面 
   2. 使用方便 比如 健康检查
   3. 一致性算法 Raft..
4. Consul的安装
   官网下载对应版本的内容，解压缩
   终端【cmd】./consul agent -dev
~~~

###### 6.3.2 Consul Java客户端开发注册中心

~~~java
1. 服务注册的本质：consul进行服务注册时，不关心服务的名字 服务的功能 做什么。ip+port 
2. 一个gprc功能的集群 定义一个逻辑的名字 
3. consul会自己进行健康检查：本质 我们提供的这个服务的ip+port是不是可以通信。
~~~

~~~java
Consul Java客户端 进行注册中心服务端的开发 

  
public class TestServer {
    public static void main(String[] args) throws IOException, InterruptedException {
        //1 模拟服务  localhost:9000
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        int port = new Random().nextInt(65535);
        serverSocketChannel.bind(new InetSocketAddress(port));

        //2 consul java client进行注册服务
        // 连接到consul服务器
        Consul consulConnection = Consul.builder().build();
        // 获得consul客户端对象
        AgentClient agentClient = consulConnection.agentClient();

        String serviceId = "Server-" + UUID.randomUUID().toString();

        // 进行服务的注册
        Registration service = ImmutableRegistration.builder()
                .name("grpc-server")
                .id(serviceId)
                .address("localhost")
                .port(port)
                .tags(Collections.singletonList("server"))
                .meta(Collections.singletonMap("version", "1.0"))
                // ttl http tcp
                // 通过tpc的方式进行健康检查
                .check(Registration.RegCheck.tcp("localhost:" + port, 10))
                .build();

        agentClient.register(service);

        Thread.sleep(100 * 1000);
    }
}
删除一个失效的服务
 http://127.0.0.1:8500/v1/agent/service/deregister/Server-b37769f1-8422-4919-b3a7-c1235d1cdc22 
健康检查
  Registration.RegCheck.tcp("localhost:" + port, 10)
  如果不做健康检查，那么consul是无法确定服务是否存活。
  底层 开了新的线程 通过延迟队列的方式 与 对应的ip+port进行通信。
~~~

~~~java
Consul Java客户端 进行注册中心客户端的开发
client根据集群的名字（服务的名字） 获得 一组健康的服务。根据负载均衡的算法，获取实际通信的rpc进行调用
  
public class TestClient {
    public static void main(String[] args) {
        Consul consulConnection = Consul.builder().build();
        HealthClient healthClient = consulConnection.healthClient();
        ConsulResponse<List<ServiceHealth>> allServiceInstances = healthClient.getHealthyServiceInstances("grpc-server");

        List<ServiceHealth> response = allServiceInstances.getResponse();
        for (ServiceHealth serviceHealth : response) {
            System.out.println("serviceHealth.getService() = " + serviceHealth.getService());
        }
    }
}

1. 获取健康实例的时候，consul有可能同步不准确。
2. 由负载均衡决定 客户端访问那个服务。【客户端负载均衡】 
~~~

###### 6.3.3 Grpc中Consul注册中心的开发

- 服务端的处理

  ~~~java
  public class GrpcServerConsul {
      public static void main(String[] args) throws InterruptedException, IOException {
          Random random = new Random();
          int port = random.nextInt(65535);
  
          ServerBuilder<?> serverBuilder = ServerBuilder.forPort(port);
          serverBuilder.addService(new HelloServiceImpl());
          Server server = serverBuilder.build();
  
          server.start();
          log.debug("server is starting......");
          registerToConsul("127.0.0.1", port);
          server.awaitTermination();
      }
  
      private static void registerToConsul(String address, int port) {
          Consul consul = Consul.builder().build();
          AgentClient client = consul.agentClient();
  
          String serviceId = "Server-" + UUID.randomUUID().toString();
  
          ImmutableRegistration service = ImmutableRegistration.builder()
                  .id(serviceId)
                  .name("grpc-server")
                  .address(address)
                  .port(port)
                  .tags(Collections.singletonList("server"))
                  .meta(Collections.singletonMap("version", "1.0"))
                  .check(Registration.RegCheck.tcp(address + ":" + port, 10))
                  .build();
  
          client.register(service);
  
      }
  }
  ~~~

- 客户端的处理

  ~~~markdown
  1. 客户端在访问具体的grpc服务之前，到命名中心（注册中心 consul）去查找注册的服务。
     命名解析 
         NameResolver 
            根据名字（服务的名字） 去注册中心查找服务，进而进行访问
         NameResolverProvider 
            作用：命名解析的提供者（工厂），用于创建命名解析对象，完成对于注册中心的访问。
     内置的命名解析 
         DNS方式处理的注册中心。修改自定义的名字解析的优先级，高于DNS才可以使用consul进行替换。
   2. 负载均衡   
         grpc客户端内置 
  ~~~

  ~~~java
  1. 自定义NameResolver
  2. 自定义NameResolverProvider
  ~~~

  - customNameResolver

    ~~~java
    @Slf4j
    public class CustomNameResolver extends NameResolver {
    
        private final ScheduledThreadPoolExecutor scheduledThreadPoolExecutor = new ScheduledThreadPoolExecutor(10);
    
        private Consul consul;
        private String authority;
        private Listener2 listener;
    
        //用于为authority赋值
        public CustomNameResolver(String authority) {
            this.authority = authority;
            consul = Consul.builder().build();
        }
    
        //authority 概念 服务器集群的名字 grpc-server
        @Override
        public String getServiceAuthority() {
            return this.authority;
        }
    
        @Override
        /*
           listener 中文 叫做监听
           start方法中我们要周期的发起对于名字解析的请求，获取最新的服务列表
         */
        public void start(Listener2 listener) {
            this.listener = listener;
            scheduledThreadPoolExecutor.scheduleAtFixedRate(this::resolve, 10, 10, TimeUnit.SECONDS);
        }
    
        //定期完成的工作 就是在resolve中处理的
        //完成解析的工作
        private void resolve() {
            log.debug("开始解析 服务.... {} ", this.authority);
            //1  获取所有的健康服务 grpc-server
            //把所有的健康服务的 ip+port封装 InetSocketAddress
            List<InetSocketAddress> addressList = getAddressList(this.authority);
    
            if (addressList == null || addressList.size() == 0) {
                log.debug("解析存在问题：{} 失败....." + this.authority);
                this.listener.onError(Status.UNAVAILABLE.withDescription("没有可用的节点...."));
                return;
            }
    
            //2 负载均衡 gprc 把所健康服务 封装 EquivalentAddressGroup {address:port}
            // InetSocketAddress ---> EquivalentAddressGroup
            List<EquivalentAddressGroup> equivalentAddressGroups = addressList.stream()
                    .map(EquivalentAddressGroup::new)
                    .collect(Collectors.toList());
    
            //3 命名解析完成 ResultionResult
            ResolutionResult resolutionResult = ResolutionResult.newBuilder()
                    .setAddresses(equivalentAddressGroups)
                    .build();
    
            //监听名字解析的结果
            this.listener.onResult(resolutionResult);
        }
    
        private List<InetSocketAddress> getAddressList(String authority) {
            HealthClient healthClient = consul.healthClient();
            ConsulResponse<List<ServiceHealth>> response = healthClient.getHealthyServiceInstances(authority);
    
            List<ServiceHealth> healthList = response.getResponse();
    
            return healthList.stream()
                    .map(ServiceHealth::getService)
                    .map(service -> new InetSocketAddress(service.getAddress(), service.getPort()))
                    .collect(Collectors.toList());
        }
    
        @Override
        public void shutdown() {
            scheduledThreadPoolExecutor.shutdown();
        }
    
        @Override
        public void refresh() {
            super.refresh();
        }
    }
    ~~~

  - CustomNameResolverProvider

    ~~~java
    public class CustomNameResolverProvider extends NameResolverProvider {
    
        @Override
        public NameResolver newNameResolver(URI targetUri, NameResolver.Args args) {
            return new CustomNameResolver(targetUri.toString());
        }
    
        @Override
        //优先级应该高于DNS（5）命名服务
        protected int priority() {
            return 6;
        }
    
        @Override
        //名字解析 （注册中心）通信的协议 consul
        public String getDefaultScheme() {
            return "consul";
        }
    
        @Override
        //作用：告知Grpc 自定义的ResolverProvider生效
        protected boolean isAvailable() {
            return true;
        }
    }
    
    ~~~

  - grpc客户端的开发

    ~~~java
    1. 原有的开发方式保留
    2. 引入自定义名字解析
    3. 引入负载均衡的算法 
    public class NameResolverGrpcClient {
        public static void main(String[] args) throws InterruptedException {
            //1 引入新的命名解析
            NameResolverRegistry.getDefaultRegistry().register(new CustomNameResolverProvider());
    
            //2客户端的开发
            ManagedChannel managedChannel = ManagedChannelBuilder.forTarget("grpc-server")
                    .usePlaintext()
                    //3 引入负载均衡算法 round_robin轮训 负载均衡
                    .defaultLoadBalancingPolicy("round_robin")
                    .build();
    
    
            //3 获得stub对象进行调用
            //模拟发送4次请求
            for (int i = 0; i < 4; i++) {
                sendRequest(managedChannel, i);
                Thread.sleep(1000);
            }
    
    
            managedChannel.awaitTermination(10, TimeUnit.SECONDS);
        }
    
        private static void sendRequest(ManagedChannel managedChannel, int idx) {
            HelloServiceGrpc.HelloServiceBlockingStub helloServiceBlockingStub = HelloServiceGrpc.newBlockingStub(managedChannel);
            HelloProto.HelloRespnose helloRespnose = helloServiceBlockingStub.hello(HelloProto.HelloRequest.newBuilder().setName("xiaohei " + idx).build());
            System.out.println("helloRespnose.getResult() = " + helloRespnose.getResult());
    
        }
    }
    遗憾：内部集群名字的拼接有问题
    ~~~


  - 服务器坑

    ~~~markdown
    在原始的gRPC与consul进行服务注册时，会产生一个Http2Connection异常。
    原因：
       进行健康汇报检查时，consul代码会通过.check(Registration.RegCheck.tcp(address + ":" + port, 10))，向grpc-server发起访问，如果这个访问没有问题，则上报consul服务器，这样就完成了健康的检测。但是在.check(Registration.RegCheck.tcp(address + ":" + port, 10))向grpc-server发请求时，站在grpc-server的角度，他会认为这是一个普通的客户端方法，他需要建立Http2连接，而consul的客户端代码，是不支持http2协议，无法建立Http2的连接，所以就抛出Http2ConnectionHandler Broken pipe。
    注意：
       1. 这个异常不会影响到我们目前程序的运行。
       2. 不能为了取消这个异常，而不使用 .check(Registration.RegCheck.tcp(address + ":" + port, 10))。
       3. gRPC+consul+springboot ---> springcloud
          健康检查 springboot actuator进行整合 而不采用.check(Registration.RegCheck.tcp(address + ":" + port, 10))。
          这样也就不会出现这个异常了。
    ~~~

###### 6.3.4 Grpc与SpringBoot整合的高级应用

- 拦截器

  ~~~java
  开发一个拦截器 
     xxxxInterceptor  implements ClientInterceptor 
     public class CustomBootInterceptor implements ClientInterceptor {
      @Override
      public <ReqT, RespT> ClientCall<ReqT, RespT> interceptCall(MethodDescriptor<ReqT, RespT> method, CallOptions callOptions, Channel next) {
          System.out.println("这是拦截器 起作用了....");
          return next.newCall(method, callOptions);
      }
  }
  2. 引入拦截器
   @Configuration
  public class CustomBootInterceptorConfiguration {
  
  
      //@Bean 让spring容器 认识这个Bean ,那Spring容器会不会把他当成grpc的拦截器呢？
      @GrpcGlobalClientInterceptor
      public CustomBootInterceptor customBootInterceptor() {
          return new CustomBootInterceptor();
      }
  }
  
  3. 第二种引入拦截器的方式
    @GrpcGlobalClientInterceptor
  public class CustomBootInterceptor implements ClientInterceptor {
      @Override
      public <ReqT, RespT> ClientCall<ReqT, RespT> interceptCall(MethodDescriptor<ReqT, RespT> method, CallOptions callOptions, Channel next) {
          System.out.println("这是拦截器 起作用了....");
          return next.newCall(method, callOptions);
      }
  }
  ~~~

- consul注册中心[grpc替换 springcloud中 openFeign]

  ~~~java
  1. 服务器端开发 【nacos,zookeeper】
     1. 依赖 
        <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-consul-discovery</artifactId>
              <version>3.1.2</version>
         </dependency>
         
         <!--健康检查  http协议 -->
         <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-actuator</artifactId>
          </dependency>
      2. application.yml
       
          spring:
            application:
              name: boot-server
            cloud:
              consul:
                discovery:
                  register: true
                  hostname: 127.0.0.1
                  port: 8500
                  heartbeat:
                    enabled: true
  
          #  main:
          #    web-application-type: none
  
  
          grpc:
            server:
              port: 9000
       
      3. 启动类
           @EnableDiscoveryClient
  
      注意事项：
            1. 进行健康检查 heartbeat.enabled = true
            2. 服务端做集群
            3. 替换注册中心（consul zk nacos）
               https://github.com/yidongnan/grpc-spring-boot-starter
               1. 替换cloud 依赖jar
               2. application.yml
                  ip+port 
                
  2. 客户端开发
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-consul-discovery</artifactId>
              <version>3.1.2</version>
          </dependency>
  
          <!--健康检查  http协议 -->
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-actuator</artifactId>
          </dependency>
          application.yml
            grpc:
              client:
                cloud-grpc-server:
                  address: 'discovery://boot-server'
                  negotiation-type: plaintext
                  enable-keep-alive: true
                  keep-alive-without-calls: true
          
          通过这个注解进行注入：
                @GrpcClient("cloud-grpc-server")
  ~~~

  



