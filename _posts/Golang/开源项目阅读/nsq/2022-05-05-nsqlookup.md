---
title: nsq源码阅读(2)-nsqlookup
category: [Golang]
tags: [Golang,源码阅读,nsq]
layout: post
---

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

nsqlookup 的服务发现是通过 HTTP 协议实现的。消费者在订阅主题时，会向 nsqlookup 发送 HTTP 查询请求，请求中包含主题名称。nsqlookup 会返回所有与该主题相关的所有 nsqd 的地址。消费者会根据返回的 nsqd 地址，通过 TCP 协议与 nsqd 节点建立连接，用于接收消息。
因此，nsqlookup需要维护nsqd实例信息的维护以及提供查询接口

nsqd实例信息的维护需要以下几个功能:
* **nsqd实例注册及更新**：当 nsqd 实例启动时，它会连接到 nsqlookupd 并通过 TCP 协议发送 `IDENTIFY` 和 `REGISTER` 命令，将自己的信息（地址、端口、版本等）以及它们正在处理的主题和通道注册到 nsqlookupd。
* **nsqd 实例注销**：当 nsqd 实例关闭时，它会通过 TCP 协议发送 `UNREGISTER` 命令，将自己以及相关的主题和通道从 nsqlookupd 注销。
* **心跳检测**：nsqd 实例会周期性地向 nsqlookupd 发送 `PING` 命令，作为心跳。这有助于 nsqlookupd 了解 nsqd 实例的健康状况，从而在 nsqd 故障时，可以通知消费者连接到其他可用的 nsqd 实例。

nsqd与nsqlookup之间的注册、注销和心跳操作是通过一个基于行的文本协议的tcp server进行的

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

nsqd和nsqdlookup会建立一个长连接，通过这个连接，nsqd 可以实时地将生产者实例信息（如新创建的主题和通道）发送给 nsqlookupd。这种实时性对于 NSQ 这样的实时消息队列系统非常重要。关于为什么使用TCP而不是HTTP，可以总结为以下几点:

1. **性能和资源消耗**：TCP 相较于 HTTP，具有较低的开销和更好的性能。HTTP 是基于 TCP 的应用层协议，它在 TCP 之上添加了额外的开销，如 HTTP 请求/响应的头部信息。而在 NSQ 的场景下，通过直接使用 TCP 协议，可以减少这些额外的开销，从而提高整体性能。此外，TCP 连接的建立和维护相对较为轻量，有助于降低资源消耗。

2. **长连接和实时性**：TCP 支持长连接，这意味着 nsqd 和 nsqlookupd 之间可以保持持久连接，而无需频繁地建立和断开连接。通过长连接，nsqd 可以实时地将生产者实例信息（如新创建的主题和通道）发送给 nsqlookupd。这种实时性对于 NSQ 这样的实时消息队列系统非常重要。

3. **双向通信**：TCP 协议支持全双工通信，这意味着 nsqd 和 nsqlookupd 可以在同一连接上同时发送和接收数据。这有助于简化通信过程，提高整体效率。

4. **简单性和可控性**：使用自定义的基于 TCP 的协议，NSQ 可以更精确地控制通信过程，同时降低了整个系统的复杂性。HTTP 协议本身具有丰富的特性和扩展性，但在 NSQ 的场景下，并不需要这些额外的功能。因此，直接使用 TCP 协议可以使整个系统更简单、易于维护和优化。

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

在nsqd与nsqlookup建立连接的时候，除了REGISTER指令，nsqd还会发送IDENTIFY指令。`IDENTIFY` 命令主要用于告知 `nsqlookupd` 关于客户端（包括 `nsqd` 节点和消费者客户端）的元数据，例如客户端版本、配置参数和支持的特性等。这些元数据对于 `nsqlookupd` 的管理和协调功能非常重要。

虽然在某种程度上，`REGISTER` 和 `IDENTIFY` 命令都包含了客户端的信息，但它们的目的和所包含的信息有所不同。`REGISTER` 命令更关注于通知 `nsqlookupd` 关于 `nsqd` 节点的主题和通道，而 `IDENTIFY` 命令关注于提供有关客户端的元数据信息。

除了在建立连接时发送 `IDENTIFY` 命令外，通常情况下不会再触发 `IDENTIFY`。然而，在某些特殊情况下，例如客户端的配置发生变化时，可能需要再次发送 `IDENTIFY` 命令以通知 `nsqlookupd` 更新元数据信息。但在实际应用中，这种情况相对较少。

#### 心跳检测

跳检测是一种监控 nsqd 节点和 nsqlookupd 之间连接状态的机制。心跳检测的主要目的是确保连接的持续性，并及时检测到节点的故障。nsqd 节点会定期向 nsqlookupd 发送心跳请求（默认间隔为 30 秒），这样 nsqlookupd 可以知道节点仍然活跃。心跳检测通过 PING 命令实现。

当 nsqlookupd 长时间没有收到来自某个 nsqd 节点的心跳时，它会认为该节点已经失去连接或出现故障。这时，nsqlookupd 会为该节点设置一个“墓碑”（tombstone）标志。墓碑标志表示 nsqd 节点已经失效，不应再被消费者客户端用于接收消息。一旦设置了墓碑标志，nsqlookupd 将不再将该 nsqd 节点包含在查询结果中。消费者客户端在请求主题生产者信息时，将不会看到已被标记为墓碑的 nsqd 节点。

设置墓碑标志有助于避免消费者客户端尝试连接已失效的 nsqd 节点，从而提高了整个 NSQ 系统的健壮性和可靠性。同时，墓碑标志可以在 nsqd 节点重新上线并重新发送心跳时自动清除，使得 nsqd 节点能够重新加入 nsqlookupd 的查询结果。这种自动恢复机制进一步提高了 NSQ 系统在面对节点故障时的弹性。

### 总结

nsqlookupd 是 NSQ 中的一个关键组件，负责管理 nsqd 节点的注册和发现。它采用了高效的事件驱动模型和长连接机制，确保了在大规模并发连接下的性能表现



