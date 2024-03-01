
## RPC框架的目的是什么
RPC框架的目标就是**让远程服务调用更加简单、透明**，RPC框架负责屏蔽底层的传输方式（TCP 或者 UDP）、序列化方式（XML/Json/ 二进制）和通信细节

## 主流的RPC框架
业界主流的 RPC 框架整体上分为三类：

* 支持多语言的 RPC 框架，比较成熟的有 Google 的 gRPC、Apache（Facebook）的 Thrift；
* 只支持特定语言的 RPC 框架，例如新浪微博的 Motan；
* 支持服务治理等服务化特性的分布式服务框架，其底层内核仍然是 RPC 框架, 例如阿里的 Dubbo。
  随着微服务的发展，基于**语言中立性原则**构建微服务，逐渐成为一种主流模式，例如对于后端并发处理要求高的微服务，比较适合采用 Go 语言构建，而对于前端的 Web 界面，则更适合 Java 和 JavaScript。
因此，基于多语言的 RPC 框架来构建微服务，是一种比较好的技术选择。例如 Netflix，API 服务编排层和后端的微服务之间就采用 gRPC 进行通信。

## RPC框架原理

## gRPC 是什么

### 简介
 gRPC 是一个高性能、开源和通用的 RPC 框架，面向服务端和移动端，基于 HTTP/2 设计。
### 支持的语言
当前支持 C、Java 和 Go 语言，其中 C 版本支持 C、C++、Node.js、C# 等。
###

gRPC 基于 HTTP/2 标准设计，带来诸如双向流、流控、头部压缩、单 TCP 连接上的多复用请求等特性。这些特性使得其在移动设备上表现更好，更省电和节省空间占用。

### grpc的特点
* 语言中立，支持多种语言；
* 基于 IDL ( 接口定义语言（Interface Define Language）)文件定义服务，通过 proto3 工具生成指定语言的数据结构、服务端接口以及客户端 Stub；
通信协议基于标准的 HTTP/2 设计，支持·双向流、消息头压缩、单 TCP 的多路复用、服务端推送等特性，这些特性使得 gRPC 在移动端设备上更加省电和节省网络流量；
* 序列化支持 PB（Protocol Buffer）和 JSON，PB 是一种语言无关的高性能序列化框架，基于 HTTP/2 + PB, 保障了 RPC 调用的高性能。

### PROTOBUF协议
#### 什么是PROTOBUF协议
protocolbuffer(以下简称PB)是google的一种数据交换的格式，它独立于语言，独立于平台。google 提供了多种语言的实现：JAVA、C++、Python，每一种实现都包含了相应语言的编译器以及库文件。由于它是一种二进制的格式，比使用 xml 进行数据交换快许多。可以把它用于分布式应用之间的数据通信或者异构环境下的数据交换。作为一种效率和兼容性都很优秀的二进制数据传输格式，可以用于诸如网络传输、配置文件、数据存储等诸多领域。

#### PROTOBUF数据协议的优劣势
优点：
* 二进制消息，性能好/效率高（空间和时间效率都很不错）
* 平台无关，语言无关，可扩展；
* 提供了友好的动态库，使用简单；
* 解析速度快，比对应的XML快约20-100倍；
* 序列化数据非常简洁、紧凑，与XML相比，其序列化之后的数据量约为1/3到1/10；
* 支持向前兼容（新加字段采用默认值）和向后兼容（忽略新加字段），简化升级；
* 支持多种语言（可以把 proto 文件看做 IDL 文件）；
* Netty 等一些框架集成。
缺点:
* 官方只支持C++，JAVA和Python语言绑定；
* 二进制可读性差；
* 二进制不具有自描述特性；
* 默认不具备动态特性（可以通过动态定义生成消息类型或者动态编译支持）；
* 只涉及序列化和反序列化技术，不涉及 RPC 功能（类似 XML 或者 JSON 的解析器）。


### grpc连接建立过程

在建立连接之前，客户端/服务端都会发送连接前言（Magic+SETTINGS），确立协议和配置项。

在传输数据时，是会涉及滑动窗口（WINDOW_UPDATE）等流控策略的。

传播 gRPC 附加信息时，是基于 HEADERS 帧进行传播和设置；而具体的请求/响应数据是存储的 DATA 帧中的。

请求/响应结果会分为 HTTP 和 gRPC 状态响应两种类型。

客户端发起 PING，服务端就会回应 PONG，反之亦可。

### grpc 调用过程

#### 服务端
- [ ] 为啥grpc服务端是tcp server呢？
##### 服务注册

* 创建连接
* 创建业务客户端实例
* 调用RPC接口

### 服务发现


## 资料

https://toutiao.io/posts/9bek5r/preview

https://segmentfault.com/a/1190000038922260

https://blog.csdn.net/fly910905/article/details/104130202

https://blog.csdn.net/Forward__/article/details/119812570

https://learnku.com/articles/33418

https://learnku.com/articles/33418

https://developer.aliyun.com/article/656330

[详细分析http2 和http1.1 区别](https://www.jianshu.com/p/63fe1bf5d445)

**P0** [思考gRPC ：为什么是HTTP/2](https://developer.aliyun.com/article/656330)
这个思考系列有点东西

## 设计目标
gRPC是google开源的高性能跨语言的RPC方案。gRPC的设计目标是在任何环境下运行，支持可插拔的负载均衡，跟踪，运行状况检查和身份验证。它不仅支持数据中心内部和跨数据中心的服务调用，它也适用于分布式计算的最后一公里，将设备，移动应用程序和浏览器连接到后端服务。
### 跨语言
定义protobuf 生成多套语言 go、python(我们的项目中有用python编写的)
### 为什么选择http2
- [ ] 什么是http2,它的栈是怎么样的
> 简而言之，gGRPC把元数据放到HTTP/2 Headers里，请求参数序列化之后放到 DATA frame里。
Headers、DATA frame
还是要小林 ！！！
- [ ] 准确来说gRPC设计上是分层的，底层支持不同的协议，目前gRPC支持 这是什么意思?
[「连载三」gRPC Streaming, Client and Server](https://eddycjy.com/posts/go/grpc/2018-09-24-stream-client-server/)

[了解 gRPC 一篇就够了](https://toutiao.io/posts/9bek5r/preview)
[gRPC系列 ：RPC 框架原理是？gRPC 是什么？gRPC设计原则](https://blog.csdn.net/fly910905/article/details/104130202)
[grpc原理及四种实现方式](https://blog.csdn.net/Forward__/article/details/119812570)
[gRPC 之流式调用原理 http2 协议分析（四）](https://learnku.com/articles/33418)
*[gRPC源码分析](https://jiacyer.com/2020/06/28/gRPC-basic/)


[【吐血整理】超全golang面试题合集+学习指南+知识图谱 涵盖大部分golang程序员所需要掌握的核心知识](https://segmentfault.com/a/1190000038922260)


*[看完这篇文章，再也不用担心面试被问 RPC](https://zhuanlan.zhihu.com/p/88597686)