[toc]

## 两日计划

- [ ] 中间件生态
- [ ] string.Builder
* gin源码 geektutu
* grpc
* 微服务实战相关内容
* [为什么这么设计](https://draveness.me/whys-the-design/)

## Golang

### 源码阅读

#### gin

- [ ] 路由匹配算法
  已知是前缀路由匹配算法，具体抽空再去了解

### 运行时

#### 并发编程

- [X] 什么时候用mutex，什么时候用channel
- [ ] GMP模型
  - [ ] 为什么要有P？每个M都弄一个本地的G队列不行吗
- [ ] 协程
  - [ ] 调度方式
    Go语言底层原理剖析中有，抓紧去看下，做总结
  - [ ] 调度策略
- [X] 协程泄露的几种情况
- [ ] **P1**网络轮询器
  epoll、select一起看，做总结
- [ ] select原理
- [X] 读写锁
- [X] 协程池
  - [X] [Handling 1 Million Requests per Minute with Golang](https://medium.com/smsjunk/handling-1-million-requests-per-minute-with-golang-f70ac505fcaa)
    [中文版](https://learnku.com/go/t/23456/using-the-go-language-to-handle-1-million-requests-per-minute)
- [ ] [为什么使用通信共享内存](https://draveness.me/whys-the-design-communication-shared-memory/)

#### 内存管理

- [ ] **P1**内存分配器
  - [ ] tcmaolloc
- [ ] 栈内存管理
- [ ] 堆内存管理
  - [ ] 堆内存结构
  - [ ] 堆内存管理数据结构
- [ ] 逃逸分析
    - [ ] **P2** 整理
  找案例
- [ ] 垃圾回收
  还有细节的东西需要了解(回收循环...)
  - [X] 三色标记
- [X] 对象池/内存池
  底层实现还要再看下
  - [X] [Go 并发编程 — 深入浅出 sync.Pool ，围观最全的使用姿势，理解最深刻的原理](https://zhuanlan.zhihu.com/p/352210244)
  - [ ] 底层实现
    这个也挺重要
- [ ] 零拷贝
  - [ ] linux 零拷贝技术
- [ ] **P0**gc优化
  - [ ] [golang gc 优化思路以及实例分析](https://my.oschina.net/u/2950272/blog/1788299)
  - [ ] **P0** [Go问题集合(gc问题都看一下)](https://www.bookstack.cn/read/qcrao-Go-Questions/spilt.14.GC-GC.md)
  - [ ] **P1** [go语言项目优化（经验之谈）](https://cloud.tencent.com/developer/article/1374180?from=article.detail.1165854)

##### 资料

- [ ] **P0**[万字长文图解 Go 内存管理分析：工具、分配和回收原理](https://mp.weixin.qq.com/s/rydO2JK-r8JjG9v_Uy7gXg)
- [ ] [golang trace](https://segmentfault.com/a/1190000019736288)
- [ ] **P1**[Golang SQL连接池梳理](https://www.cnblogs.com/zhuchangwu/p/13412853.html#%E4%B8%80%E5%A6%82%E4%BD%95%E7%90%86%E8%A7%A3%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5)

### 基础知识

- [ ] channel 是否用到了锁
- [ ] **P2**go与php的比较
- [ ] **P2**IO包
- [ ] **P1**[面试官：两个nil比较结果是什么？](https://mp.weixin.qq.com/s/CNOLLLRzHomjBnbZMnw0Gg)

#### 资料

- [ ] [深入介绍 Golang 中的 bufio.Scanner](https://zhuanlan.zhihu.com/p/37673679)
- [ ] 关键字
  - [ ] go
  - [ ] **P0**defer、panic、recover
    defer 面试题
- [ ] 数据结构
  - [ ] channel
  - [ ] context#### 资料

    - [ ] [context 如何被取消](https://golang.design/go-questions/stdlib/context/cancel/)
  - [ ] map

    - [ ] 如何高效使用并发
- [ ] 语言基础
  - [ ] make 和 new 的区别
  - [ ] 深拷贝和浅拷贝
  - [ ] 变量比较 == nil
  - [X] struct能否比较
    - [X] [Go 面试官：Go 结构体是否可以比较，为什么？](https://segmentfault.com/a/1190000040099215)

### 性能优化

- [X] [节约千台容器，Go程序性能瓶颈实战分析](https://zhuanlan.zhihu.com/p/490652454)
  好像没啥东西
  总结下

### 网络编程

- [ ] 网络编程
  - [ ] socket编程
    - [ ] 超时处理
    - [ ] 发送和接受数据
      - [ ] 错误处理
- [ ] https://golang.org/pkg/net/#OpError/

#### 资料

- [ ] [史上最详细Go语言之网络编程介绍（干货！）](https://zhuanlan.zhihu.com/p/302547547)
- [ ] [GO的网络编程分享](https://learnku.com/articles/57947)
  这两个网络编程没什么东西，找时间总结一下
- [ ] [网络编程实战](https://time.geekbang.org/column/intro/100032701?tab=catalog)
- [ ] pprof

### 资料

- [ ] [Go常见面试题【由浅入深】2022版](https://zhuanlan.zhihu.com/p/471490292)
- [ ] [ 如何使用 Go 更好地开发并发程序？](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=287#/detail/pc?id=3805)

## 项目设计

- [ ] [为什么这么设计](https://draveness.me/whys-the-design/)
- [ ] 需求到落地的过程
- [ ] [IM系统设计](https://time.geekbang.org/column/article/494460)
- [ ] [如何设计一个秒杀系统](https://time.geekbang.org/column/article/40153)
- [ ] [高并发系统设计 40 问](https://time.geekbang.org/column/intro/100035801?tab=catalog)

## 网络

- [ ] [终于搞懂了服务器为啥产生大量的TIME_WAIT！](https://segmentfault.com/a/1190000040786792)
- [ ] [TIME_WAIT and Its Design Implications](https://segmentfault.com/a/1190000041796954?utm_source=sf-similar-article)
- [ ] netstat命令
- [ ] **P0** 为什么TCP挥手要4次
  因为在挥手的时候，收到对方的结束请求需要一次ack，自己结束的时候需要一次fin，这两次请求一般是需要分开发送的

## 操作系统

- [ ] **P0** [Linux 内核技术实战课](https://time.geekbang.org/column/article/289896)

### IO多路复用

- [ ] **P0** epoll原理 - [ ] 两种触发机制 ## 分布式 ### 分布式计算 ### 分布式存储 - [ ] 分片 - [ ] 复制 - [ ] 主从复制原理 - [ ] 事务 - [ ] 一致性与共识 ## 微服务 - [ ] 如何划分微服务
- [ ] 分布式理论
- [ ] **P1** 分布式链路追踪
  如何实现，在你们的项目中是怎么做的
  - [ ] OpenTracing 分布式链路追踪规范
    - [ ] [34 | OpenTracing 规范介绍与分布式链路追踪组件选型](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=287#/detail/pc?id=3829)
    - [ ] [Jaeger文档](https://www.jaegertracing.io/docs/1.35/)
- [ ] 全链路压测
- [ ] 分布式日志系统
- [ ] 分布式锁
  - [ ] 如何解决网络问题带来的死锁问题
- [ ] 分布式事务
- [ ] 服务注册
- [ ] 配置中心
  - [ ] 配置中心选型
  - [ ] apollo
- [ ] 负载均衡
  - [X] 算法
  - [ ] 数据分片
  - [ ] **P0**[28 | 案例：如何在 Go 微服务中实现负载均衡？](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=287#/detail/pc?id=3823)
- [ ] 持续集成

### 资料

- [ ] [11 | 案例：如何结合 Jenkins 完成持续化集成和自动化测试？](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=287#/detail/pc?id=3809)
- [ ] [周志明的软件架构课](https://time.geekbang.org/column/article/309727)

### 网关

- [ ] 反向代理

#### 资料

- [ ] [Go 简单而强大的反向代理（Reverse Proxy）](https://h1z3y3.me/posts/simple-and-powerful-reverse-proxy-in-golang/)

#### 什么是微服务网关

#### 网关选型

##### Nginx

- [ ] nginx 工作模型
- [ ] nginx 事件处理模型

###### 资料

- [ ] [Nginx为什么这么高效？看完你就明白Nginx核心原理](https://zhuanlan.zhihu.com/p/129020819)

##### traefik

- [ ] [traefik 官方文档](https://doc.traefik.io/traefik/)

### 高可用

- [ ] CAP理论
  - [X] 基本知识
  - [ ] 一致性级别
  - [ ] AP和CP怎么选择
- [ ] 冗余设计
- [ ] 熔断设计
- [ ] 如何做到滚动发布，灰度发布？
  - [ ] [基于 Nginx 实现灰度发布与 AB 测试](https://segmentfault.com/a/1190000040786818?utm_source=sf-similar-article)

### 其他

- [ ] 监控
- [ ] 认证
- [ ] 微服务与云原生
  - [ ] [万字长文揭穿你，根本就不懂云原生！ ](https://www.sohu.com/a/483214774_411876)
    泛泛而谈的东西
- [ ] 演进

## 数据库

- [ ] 什么是OLTP/OLAP数据库### 资料

  - [ ] [1. 什么是OLTP、OLAP、数据库和数据仓库？](https://zhuanlan.zhihu.com/p/166408697)
- [ ] Mysql & Nosql & 时序数据库
- [ ] **P1**数据库连接池是怎么实现的
- [ ] [跨系统数据一致性问题经验实战](https://gitchat.cn/books/627627e35bef09a6a51a9a18/index.html)
  基础知识

### Mysql

#### 基础

- [ ] left join 、inner join、right join
- [ ] 有哪些索引类型
  - [X] [MySQL索引都有哪些分类？](https://developer.huawei.com/consumer/cn/forum/topic/0204405591412170236)
    似乎没什么用
- [ ] 建立索引的依据
- [ ] 什么时候会索引失效
- [ ] 死锁问题

#### 集群

- [ ] 主从复制是怎么实现的

### Mongo

- [ ] 场景
- [ ] 索引与Mysql有什么不同#### 资料

  - [ ] [mongoDB 高手课](https://time.geekbang.org/course/detail/100040001-193615)

### ES

- [ ] 场景
- [ ] 优势

#### 资料

- [ ] [Elasticsearch 核心技术与实战](https://time.geekbang.org/course/detail/100030501-102666)
- [ ] [ElasticSearch在项目中具体怎么用？](https://www.zhihu.com/question/469207536/answer/2550197723)

## Docker

<!-- - [ ] -->

## 消息中间件

### Kafka

### Redis

- [ ] **P1** 缓存数据库一致性

#### 数据结构

- [ ] zsort 能存什么?
- [ ] 用过哪些数据结构
- [ ] Redis面试题
- [ ] stream的用法

#### **P1**集群

也不是很难的东西，再去看看吧

## **P2** K8S

抓紧刷一遍

### 基本认识

- [ ] K8S是什么，由哪几部分构成
- [ ] **P1** POD是什么

#### 资料

- [ ] [Kubernetes 入门实战课](https://time.geekbang.org/column/article/528554?utm_term=zeusY3O43&utm_source=geektime&utm_medium=summary&utm_campaign=100114501&utm_content=articlebottom)
- [ ] [10 | 案例：微服务 Docker 容器化部署和 Kubernetes 容器编排](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=287#/detail/pc?id=3808)

## RPC

- [ ] 有HTTP为什么还需要RPC

### Grpc

- [ ] grpc 原理
- [ ] grpc 优势
- [ ] **P1** stream 是怎么一个tcp连接能xxx的

## 设计模式

## 算法

- [ ] **P1** 滑动窗口
- [ ] **P1** 深搜和广搜

## 项目总结

- [ ] mPaaS
- [ ] 网关
- [ ] **P2** TDD和DDD怎么在实际项目中推进
- [ ] **P1** 分库分表的看法

## 技术视野

- [ ] 开源社区的运作方式
- [ ] 边缘计算
- [ ] 大厂是怎么上线一个项目的
- [ ] OpenPolicy
- [ ] flink
- [ ] hadoop
- [ ] unicode/utf8/acill
  - [ ] golang怎么解决乱码问题

### 资料

- [ ] [五分钟零基础搞懂Hadoop](https://zhuanlan.zhihu.com/p/20176725)

## 学习法

## 编辑器

[neovim](https://zhuanlan.zhihu.com/p/511992981 "lsp+lua")
