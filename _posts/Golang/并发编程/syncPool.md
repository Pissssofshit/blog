[TOC]

## 基本用法
get
New
## 底层实现
TODO
## 小结
利用 GMP 的特性，为每个 P 创建了一个本地对象池 poolLocal，尽量减少并发冲突。
每个 poolLocal 都有一个 private 对象，优先存取 private 对象，可以避免进入复杂逻辑。
在 Get 和 Put 期间，利用 pin 锁定当前 P，防止 goroutine 被抢占，造成程序混乱。
在获取对象期间，利用对象窃取的机制，从其他 P 的本地对象池以及 victim 中获取对象。
充分利用 CPU Cache 特性，提升程序性能。

## 应对面试
尽量复用对象达到...

## 资料
[深度分析 Golang sync.Pool 底层原理](https://www.cyhone.com/articles/think-in-sync-pool/)