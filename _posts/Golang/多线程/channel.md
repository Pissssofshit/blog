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


## 
当我第一次使用Go中的通道(Channels)的时候，我误以为把Channels当作一种数据结构。我将Channels看作为队列来在goroutines之间提供同步访问。 这种概念性的误解理解使我编写了许多糟糕而复杂的并发代码。

而随着时间的推移以及对Go语言的深入理解，我逐渐关注到它的行为特性。 所以现在谈到Channels，我就会想到通信(signaling)。 一个通道允许一个goroutine向另一个关于特定事件的goroutine进行通信。把Channels视为通信机制，将允许你使用定义明确且能编写出更加优雅的代码。

要了解通信是如何工作，我们必须了解它的三个属性：
* 可靠发送
* 状态
* 带/不带数据

## channel的数据收发在什么情况会阻塞？
* 为nil的情况下收发都会阻塞
* 没有缓存区的读写或者缓存区已满的写或者缓存区为空的读


## channel 关闭原则
在使用Go channel的时候，一个适用的原则是不要从接收端关闭channel，也不要关闭有多个并发发送者的channel。

换句话说，如果sender(发送者)只是唯一的sender或者是channel最后一个活跃的sender，那么你应该在sender的goroutine关闭channel，从而通知receiver(s)(接收者们)已经没有值可以读了。维持这条原则将保证永远不会发生向一个已经关闭的channel发送值或者关闭一个已经关闭的channel。




select的运行机制是怎样的？
nil channel收发数据是什么结果？


https://golang.design/go-questions/channel/read-on-close/

实际应用
https://www.s0nnet.com/archives/go-channels-behavior
https://www.s0nnet.com/archives/go-channels-practice
实现原理
https://mp.weixin.qq.com/s/40uxAPdubIk0lU321LmfRg