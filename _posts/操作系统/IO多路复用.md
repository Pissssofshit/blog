[TOC]

## I/O多路复用
要完成对I/O流的复用需要回答如下几个问题：
1.用户态怎么将文件句柄传递到内核态？
2.内核态怎么判断I/O流可读可写？
3.内核怎么通知监控者有I/O流可读可写？
4.监控者如何找到可读可写的I/O流并传递给用户态应用程序？
5.继续循环时监控者怎样重复上述步骤？

## epoll
### 是什么
epoll是一种I/O事件通知机制，是linux 内核实现IO多路复用的一个实现。
IO多路复用是指，在一个操作里同时监听多个输入输出源，在其中一个或多个输入输出源可用的时候返回，然后对其的进行读写操作。

## 资料
[彻底搞懂epoll高效运行的原理](https://www.jianshu.com/p/31cdfd6f5a48)
[epoll原理详解（最清晰）](https://blog.csdn.net/lyztyycode/article/details/79491419)
[linux五种IO模型与事件驱动模型 ](https://www.cnblogs.com/Yunya-Cnblogs/p/13246517.html)
[带你彻底理解Linux五种I/O模型](https://wiyi.org/linux-io-model.html)
[epoll在Golang的应用](https://zhuanlan.zhihu.com/p/344581947)
[Go netpoller 原生网络模型之源码全面揭秘](https://mp.weixin.qq.com/s?__biz=MzAxMTA4Njc0OQ==&mid=2651443085&idx=3&sn=2c1ed8474bc7fed68b519ce9e5f5e0b0&scene=21#wechat_redirect)