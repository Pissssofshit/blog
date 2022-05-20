
##  mechanics and semantics behind writing multithreaded code in Go

## Scheduler Semantics
### 什么是 Scheduler Semantics
#### 
P和M代表了计算机资源或是Go程序运行的上下文
P代表逻辑上的处理器，M代表操作系统线程
> This (logical processor) P and (an operating system thread) M represent the compute power or execution context for running the Go program

初始化一个G被创建出来用于管理M/P的选择(?)(或者说初始化一个G用于负责GMP的调度)，M负责指令在硬件上的运行
这样的设计，等于在操作系统之上做了一层抽象，将运行控制转移到了应用层
>This creates a new layer of abstraction above the operating system, but it moves execution control to the application level

> Go scheduler sits on top of the operating system scheduler
### Scheduler Semantics 要解决什么问题
### Scheduler Semantics 是怎么解决这个问题的
### 拓展