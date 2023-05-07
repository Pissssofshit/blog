```
title: nsq源码阅读(2)-nsqlookup
category: [Golang]
layout: post
tags: [Golang,源码阅读,nsq]
```

### 介绍

nsqlookup是 NSQ 的服务发现组件，负责维护所有 nsqd 实例的信息。生产者和消费者可以查询 nsqlookupd 以找到与特定主题相关的 nsqd 实例。当新的 nsqd 实例启动或关闭时，它们会向 nsqlookupd 注册或注销自己，以便消费者能够发现它们。

### nsqlookup

nsqlookup分别实现了http server和tcp server。

http server主要用于管理和查询操作，以下是一些主要的HTTP API:

- `/ping`：用于检查 nsqlookupd 是否运行正常。
- `/info`：返回 nsqlookupd 的版本和其他一些元数据。
- `/topics`：返回所有已注册的主题列表。
- `/channels?topic=<topic>`：返回指定主题下的所有通道列表。
- `/lookup?topic=<topic>`：根据指定的主题返回相关的 nsqd 节点信息（包括地址、端口）。

tcp server 实现了一个基于行的文本协议，主要用于 nsqd 实例与 nsqlookupd 之间的注册、注销和心跳操作。主要包括以下几种命令：

- `IDENTIFY`：nsqd 向 nsqlookupd 发送自己的身份信息（包括地址、端口、版本等）。
- `REGISTER`：nsqd 向 nsqlookupd 注册一个主题和通道。
- `UNREGISTER`：nsqd 向 nsqlookupd 注销一个主题和通道。
- `PING`：nsqd 向 nsqlookupd 发送心跳。

下面分别介绍这些功能是怎么具体实现的

### 服务发现

服务发现分为两块，nsqd实例信息的维护和信息的查询

* nsqd实例信息的维护
  * **nsqd实例注册**：当 nsqd 实例启动时，它会连接到 nsqlookupd 并通过 TCP 协议发送 `IDENTIFY` 和 `REGISTER` 命令，将自己的信息（地址、端口、版本等）以及它们正在处理的主题和通道注册到 nsqlookupd。
  * **nsqd实例更新**：当 nsqd 实例启动时，它会连接到 nsqlookupd 并通过 TCP 协议发送 `IDENTIFY` 和 `REGISTER` 命令，将自己的信息（地址、端口、版本等）以及它们正在处理的主题和通道注册到 nsqlookupd。
    - [ ] 什么时候nsqd会进行IDENTIFY
  * **nsqd 实例注销**：当 nsqd 实例关闭时，它会通过 TCP 协议发送 `UNREGISTER` 命令，将自己以及相关的主题和通道从 nsqlookupd 注销。
  * **心跳检测**：nsqd 实例会周期性地向 nsqlookupd 发送 `PING` 命令，作为心跳。这有助于 nsqlookupd 了解 nsqd 实例的健康状况，从而在 nsqd 故障时，可以通知消费者连接到其他可用的 nsqd 实例。
* 信息的查询

上文说到过，nsqd与nsqlookup之间的注册、注销和心跳操作是通过一个基于行的文本协议的tcp server进行的

```
// nsqlookupd/lookup_protocol_v1.go
func (p *LookupProtocolV1) IOLoop(c protocol.Client) error {
		var err error
	var line string

	client := c.(*ClientV1)

	reader := bufio.NewReader(client)
	for {
		line, err = reader.ReadString('\n')
		if err != nil {
			break
		}

		line = strings.TrimSpace(line)
		params := strings.Split(line, " ")

		var response []byte
		response, err = p.Exec(client, reader, params)
		// 省略...
	}
}

```

nsqd和nsqdlookup会建立一个长连接，通过这个连接，nsqd 可以实时地将生产者实例信息（如新创建的主题和通道）发送给 nsqlookupd。这种实时性对于 NSQ 这样的实时消息队列系统非常重要。

在参数经过解析之后，执行相应的命令，接下来会逐个解析

```
func (p *LookupProtocolV1) Exec(client *ClientV1, reader *bufio.Reader, params []string) ([]byte, error) {
	switch params[0] {
	case "PING":
		return p.PING(client, params)
	case "IDENTIFY":
		return p.IDENTIFY(client, reader, params[1:])
	case "REGISTER":
		return p.REGISTER(client, reader, params[1:])
	case "UNREGISTER":
		return p.UNREGISTER(client, reader, params[1:])
	}
	return nil, protocol.NewFatalClientErr(nil, "E_INVALID", fmt.Sprintf("invalid command %s", params[0]))
}
```

#### 注册

```
func (p *LookupProtocolV1) REGISTER(client *ClientV1, reader *bufio.Reader, params []string) ([]byte, error) {
	// ... 参数校验省略
	topic, channel, err := getTopicChan("REGISTER", params) //从参数中解析出topic和channel
	if err != nil {
		return nil, err
	}
	// 分别注册channel和topic
	if channel != "" {
		key := Registration{"channel", topic, channel} 
		if p.nsqlookupd.DB.AddProducer(key, &Producer{peerInfo: client.peerInfo}) {
			p.nsqlookupd.logf(LOG_INFO, "DB: client(%s) REGISTER category:%s key:%s subkey:%s",
				client, "channel", topic, channel)
		}
	}
	key := Registration{"topic", topic, ""}
	if p.nsqlookupd.DB.AddProducer(key, &Producer{peerInfo: client.peerInfo}) {
		p.nsqlookupd.logf(LOG_INFO, "DB: client(%s) REGISTER category:%s key:%s subkey:%s",
			client, "topic", topic, "")
	}

	return []byte("OK"), nil
}
```

注册会分别将channel和topic信息注册到DB中去，DB主要是一个map结构

```
type RegistrationDB struct {
	sync.RWMutex
	registrationMap map[Registration]ProducerMap
}
```

可以看到，注册到DB的数据都是存储在一个key为```Registration``` 的map中。这个结构体包含三个字段

```
type Registration struct {
	Category string
	Key      string
	SubKey   string
}
```

回到上文可以看到，Category为"channel"或"topic",key是topic的值，subkey为channel的值或者为空

在查找的时候，可以根据传参精确匹配Category、Key、SubKey，也支持模糊匹配。

而ProducerMap就是对应的生产者，包含了nsqd实例的地址、端口、版本等信息。



### Q&A

#### 为什么注册、注销、更新使用tcp协议而不是http协议

1. **性能和资源消耗**：TCP 相较于 HTTP，具有较低的开销和更好的性能。HTTP 是基于 TCP 的应用层协议，它在 TCP 之上添加了额外的开销，如 HTTP 请求/响应的头部信息。而在 NSQ 的场景下，通过直接使用 TCP 协议，可以减少这些额外的开销，从而提高整体性能。此外，TCP 连接的建立和维护相对较为轻量，有助于降低资源消耗。

2. **长连接和实时性**：TCP 支持长连接，这意味着 nsqd 和 nsqlookupd 之间可以保持持久连接，而无需频繁地建立和断开连接。通过长连接，nsqd 可以实时地将生产者实例信息（如新创建的主题和通道）发送给 nsqlookupd。这种实时性对于 NSQ 这样的实时消息队列系统非常重要。

3. **双向通信**：TCP 协议支持全双工通信，这意味着 nsqd 和 nsqlookupd 可以在同一连接上同时发送和接收数据。这有助于简化通信过程，提高整体效率。

4. **简单性和可控性**：使用自定义的基于 TCP 的协议，NSQ 可以更精确地控制通信过程，同时降低了整个系统的复杂性。HTTP 协议本身具有丰富的特性和扩展性，但在 NSQ 的场景下，并不需要这些额外的功能。因此，直接使用 TCP 协议可以使整个系统更简单、易于维护和优化。

#### 优雅的停止、重启服务

如果你看过源码，也许会对以下这个写法感到奇怪
```
func (l *NSQLookupd) Main() error {
	exitCh := make(chan error)
	var once sync.Once
	exitFunc := func(err error) {
		once.Do(func() {
			if err != nil {
				l.logf(LOG_FATAL, "%s", err)
			}
			exitCh <- err
		})
	}

	l.waitGroup.Wrap(func() {
		exitFunc(protocol.TCPServer(l.tcpListener, l.tcpServer, l.logf)) // 初始化方法
	})
	httpServer := newHTTPServer(l)
	l.waitGroup.Wrap(func() {
		exitFunc(http_api.Serve(l.httpListener, httpServer, "HTTP", l.logf))
	})

	err := <-exitCh
	return err
}
```

这个 `TCPServer` 函数的主要目的是在一个永不退出的循环中处理传入的 TCP 连接。然而，`sync.WaitGroup` 的使用主要是为了确保在某些情况下，当服务器停止接受新的连接时，仍然能正确地处理和关闭已经建立的连接。

例如，在某些情况下，您可能需要手动停止服务器，例如进行优雅的关闭或重新加载配置。在这种情况下，您需要关闭 `listener`，这将导致 `Accept()` 方法返回一个错误，并使循环退出。然而，在这种情况下，您不希望立即结束程序，因为仍然可能有一些处理中的连接。

`sync.WaitGroup` 的作用是确保所有已经启动的 `handler.Handle()` goroutine 都能在 `TCPServer` 函数返回之前完成执行。当您关闭 `listener` 时，`for` 循环会退出，但在返回之前，`wg.Wait()` 会阻塞，直到所有处理中的连接都已完成。

这种设计可以确保在服务器停止接受新连接时，不会丢失任何已经建立的连接。这有助于确保在进行优雅关闭或重新加载配置等操作时，服务器能够正确地处理和关闭所有连接。
