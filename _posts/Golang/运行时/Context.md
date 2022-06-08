## Context 是什么
```
type Context interface {
	Deadline() (deadline time.Time, ok bool)
	Done() <-chan struct{}
	Err() error
	Value(key interface{}) interface{}
}
```
子Goroutine通过监听Done() 返回的channel 来判断Context是否已超时/取消
## 干什么
* 在 Goroutine 构成的树形结构中对信号(特定数据、取消信号以及处理请求的截止日期)进行同步以减少计算资源的浪费 
## 怎么做
1. 接口返回一个只读的channel, 所有相关函数监听此channel,一旦channel关闭，所有监听者都能收到

## 资料
[context 如何被取消](https://golang.design/go-questions/stdlib/context/cancel/)