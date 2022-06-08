[TOC]

- [ ] channel 分配在堆上还是在栈上
- [ ] channel 是怎么实现并发安全的
### 数据结构
缓存数据用循环链表实现，在操作链表的时候需要加锁。
CSP即利用channel 把数据从一端拷贝到了另一端

### channel 阻塞是怎么做的
当发生阻塞时，G主动调用Go的调度器，让G进入waiting状态，并让出M,让其他g使用
同时G会被抽象成G指针和send元素的sudog保存到hchan的sendq中等待被唤醒


## 使用总结
发生panic的三种情况:
* 向一个关闭的channel进行写操作
* 关闭一个nil的channel
* 关闭一个已经关闭的channel
发生阻塞的两种情况:
* 读写一个为nil的channel
* 没有数据的channel
读取一个已经关闭的channel:读到对应类型的零值

鉴于上面发生panic的情况，优雅关闭channel可以总结为:
因为不能对已经关闭的channel进行写操作，所以关闭操作需要由发送方进行(接收方进行则发送方无法感知)，在一写一读或多读的情况下直接由发送方关闭就可以。
因为不能重复关闭channel,所以在多写的情况下，需要增加一个传递关闭信号的channel(接受者关闭作为信号的channel即可),顺带一提，多个发送者退出不再使用channel之后，不用手动关闭channel,gc会把没有引用的channel关闭的。
多发多读的情况下，为了避免多次关闭信号channel,需要加一个中间者，接受到信号之后关闭stopChan然后就退出，避免多次关闭。


## channel数据结构

```
// runtime/chan.go
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}
```
## sudog 数据结构



## channel 关闭原则
在使用Go channel的时候，一个适用的原则是不要从接收端关闭channel，也不要关闭有多个并发发送者的channel。

换句话说，如果sender(发送者)只是唯一的sender或者是channel最后一个活跃的sender，那么你应该在sender的goroutine关闭channel，从而通知receiver(s)(接收者们)已经没有值可以读了。维持这条原则将保证永远不会发生向一个已经关闭的channel发送值或者关闭一个已经关闭的channel。



select的运行机制是怎样的？
nil channel收发数据是什么结果？

## 资料
[Go 程序员面试笔试宝典](https://golang.design/go-questions/channel/read-on-close/)
[深入理解Go语言的Channels特性-没看懂](https://www.s0nnet.com/archives/go-channels-behavior)
[Go语言之Channels实际应用](https://www.s0nnet.com/archives/go-channels-practice)
- [x] [图解Go的channel底层原理](https://mp.weixin.qq.com/s/40uxAPdubIk0lU321LmfRg)