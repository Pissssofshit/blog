* 数据库查询优化
- 尽量利用索引
    - 避免比如复合索引不包含前置列
- 不走索引的情况
    - 隐式类型转换造成不使用索引
    -  order by 条件要与where中条件一致，否则order by不会利用索引进行排序
- 避免索引失效的情况
    - 多个模糊匹配
        - 交互优化
            - 给前端提供列表接口，使其提供精确的值
- 表设计优化
    - 列表的显示的值尽量都在索引里避免回表
    - 不能优化的列表则采用大宽表(只读不写的数据)
- count(*)问题
    - 大宽表（读多写少)
- in n 个id的问题
    - 没有很好的解决方法，升配置
    - exist?
- 跨库联表问题
    - 有点复杂，不管
- 页码跳页性能(*)
## 业务上的优化
1、限制用户分页，不允许进行过大的分页。比如说百度的分页，它最多只能跳转到一定页数，用户再往下翻页也只显示最后一页。

2、禁用count(*)，不去查询总条数，不计算总页数，不设计跳页的功能
# sql层面优化方式
1. 索引覆盖，延时关联
> 我们可以使用 SELECT id FROM emp ORDER BY id LIMIT 5000000,10 充当子查询，这样就只在索引中查询id，而不用回表查；然后根据子查询得出来的id集合再去查对应的select * 数据即可;


### join
* Simple Nested-Loop Join
* Block Nested-Loop Join
* Index Nested-Loop Join
#### Simple Nested-Loop Join
当被驱动表的join字段没有索引时，联表就等于驱动表的记录去扫码被驱动表的全表去匹配记录
#### Block Nested-Loop Join
加了一块内存区域，将驱动表加载到内存里，因为是内存操作，效率就比Simple算法高



- 事务的实现原理
- B 树 和 B+ 树的区别，为什么 mysql 要用 B+ 树，mongodb 要用 B 树。
- Mysql 集群如何保证数据的一致性。分别回答了弱一致性和强一致性。
- mysql的常见引擎区别
- mysql索引失效场景
#### 资料
https://mp.weixin.qq.com/s/tCXGRbSs7BgjqXf6O-8uaA
* Redis
- redis setnx + expire 有什么缺点，如何优化。因为之前自己用过，所以答上来了，但是好像不是面试官想要的最优解。
- redis各种数据结构可以做什么
- redis的持久化方式，区别？持久化过程时如何保证不会出现新的写覆盖数据?
- redis主从复制？
- 系统设计：如何实现排行榜？我的回答提到了分布式zset然后汇总
* grpc原理
    - 先往后推一下
* Golang面试题
- golang中new和make的区别
- 介绍下go的chan,chan可以做什么
- 你们的项目如何实现限流器，请用chan实现一种限流器，也可以不用chan实现
- Context 
- select 和 epoll
- GMP调度模型
- I/O多路复用中select/poll/epoll的区别？
* HTTP
- 请详细描述三次握手和四次挥手的过程，要求熟悉三次握手和四次挥手的机制，要求画出状态图
- 四次挥手中TIME_WAIT状态存在的目的是什么
- TCP是通过什么机制保障可靠性的？（从四个方面进行回答，ACK确认机制、超时重传、滑动窗口以及流量控制，深入的话要求详细讲出流量控制的机制。）
- 网络IO模型有哪些？（5种网络I/O模型，阻塞、非阻塞、I/O多路复用、信号驱动IO、异步I/O）
- HTTP报文格式
- HTTP状态码
- TCP 拥塞控制
- 描述HTTPS和HTTP的区别
- get post有什么区别
- 抓包post和get有什么区别
- 打开一个 URL 的过程。
https://www.zhihu.com/question/28586791
* ES
- 怎么用ES实现查找功能
* 鉴权算法
* 领域驱动设计(话术)
* 测试驱动开发(话术)
- 总结写下来!
* 代码规范(工具)
* Redis
* 中间件
- memcache、kafka(原理)
* Docker
- dockerfile
* 操作系统
- 线程进程协程区别
- 协程数据结构
- 用过哪些锁，自旋锁和互斥锁有什么区别。（同学们锁一定好好复习，也是必考点之一）
- 用过哪些分布式锁。答了 mysql，redis， zookeeper 分别聊了一下优缺点。
* 系统设计
- 类似于设计一个长链接转短链接，不过需要考虑高并发，回答了分库分表。
* 微服务
- 服务治理（限流、降级、熔断）
* 算法题
- 算法题一个数字比如452484515124845157515，可以去除n个数（比如1234去掉3个数 最大值就是4），求最大值
- 会议有时间段，有很多会议，求最少需要多少会议室
- 树的非递归先序遍历
- 回行矩阵遍历
- 二叉树多个节点的最近公共祖先
* 线上遇到的问题
现象: 内存和CPU占用飙高，多次OOM
排查: 
1. kibana捞OOM前日志，发现有请求花费时间很长，把请求链接拿出来
2. 接入了Pythxx，定位耗时与内存占用异常的代码段（具体函数与耗时操作，string make操作)
3. 

## Golang 写屏障是怎么实现的

## 路线
* Go语言基础
书籍:《The Go Programming Language》
* Go语言进阶
* 微服务
* 系统设计
* 项目经验
* 面试
## 资料
[Go 微服务实战 38 讲](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=287#/detail/pc?id=3812)
[手把手带你写一个 Web 框架](https://time.geekbang.org/column/article/418283)
[Go工程师](https://time.geekbang.org/learning/path-detail/47)
《Ultimate Go Notebook》
《The Go Programming Language》
[golang面试题：对未初始化的的chan进行读写，会怎么样？为什么？](https://zhuanlan.zhihu.com/p/149796956)
[不要再问我 in，exists 走不走索引了](https://segmentfault.com/a/1190000023825926)
[Golang：一文解决Map并发问题](https://cloud.tencent.com/developer/article/1539049)
[gRPC 长连接在微服务业务系统中的实践](https://www.infoq.cn/article/cpxr35bwjttgncltyekz)
[Mysql中join的那些事](https://mp.weixin.qq.com/s/tCXGRbSs7BgjqXf6O-8uaA)
[聊聊 MySQL 的优化思路](https://mp.weixin.qq.com/s/9XKYvA5hmwictkbd42w-Ug) => 输出自己的mysql优化思路
[技术分享 | MySQL 优化：JOIN 优化实践](https://zhuanlan.zhihu.com/p/103661924)
[为什么 TCP 建立连接需要三次握手](https://mp.weixin.qq.com/s/XGpIjrnxAAuHa8EVSotBQw)
[mysql实战45讲](https://funnylog.gitee.io/mysql45/)
[Golang 读写锁设计](https://segmentfault.com/a/1190000040406605)
[Golang 之 struct能不能比较](https://juejin.cn/post/6881912621616857102)
[2022Go后端开发大厂面试题](file:///Users/xiaoshuaihuang/Downloads/2022Go%E5%90%8E%E7%AB%AF%E5%BC%80%E5%8F%91%E5%A4%A7%E5%8E%82%E9%9D%A2%E8%AF%95%E9%A2%98.pdf)
[使用 jwt 完成 sso 单点登录](https://ld246.com/article/1536130162416)
[手把手教你，使用JWT实现单点登录](https://zhuanlan.zhihu.com/p/141065758)
[看完这篇文章，再也不用担心面试被问 RPC](https://zhuanlan.zhihu.com/p/88597686) <= 这类似乎得系统学习，学简单则无用
[Handling 1 Million Requests per Minute with Golang](https://medium.com/smsjunk/handling-1-million-requests-per-minute-with-golang-f70ac505fcaa)
[gRPC源码分析](https://jiacyer.com/2020/06/28/gRPC-basic/)
[全网最通俗易懂的Kafka入门！](https://zhuanlan.zhihu.com/p/95215691)
[Network Programming with Go](file:///Users/xiaoshuaihuang/Downloads/Adam%20Woodbeck%20-%20Network%20Programming%20with%20Go_%20Learn%20to%20Code%20Secure%20and%20Reliable%20Network%20Services%20from%20Scratch%20(2021,%20No%20Starch%20Press)%20-%20libgen.li.pdf)
[RPC 实战与核心原理](https://time.geekbang.org/column/intro/100046201?tab=catalog)

[性能工程](https://time.geekbang.org/column/article/355982)