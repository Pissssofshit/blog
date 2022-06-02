## 节点状态
* Follower
不直接处理client 请求
* Candidate
* Leader
会向Follower节点发送心跳消息，还会处理客户端的请求，并将客户端的更新操作以消息的形式发送到集群中的所有Follower节点。当Follower节点收到这些消息之后，会向Leader节点返回相应的相应消息。xxxx之后，会对客户端的请求进行应答。最后，Leader会提交客户端的更新操作，该操作会xxxxx

集群中的节点初始状态是Follower，
每个节点维护一个随机长度的选举超时时间，并维护一个term值。
当超过这个时长没有收到Leader 节点发送的心跳消息之后。
节点会转化为Candidate节点，并向其他节点发送选举请求。
Candidate 节点若接受到其他term值更大的选举请求，会转化为Follower 状态（比如选举时间较长，选举还未完毕就已有节点超时，开始了下一轮选举,所以Raft算法要求选举时间远小于广播时间)
在获取到集群中半数以上的选票之后，Candidate成为Leader 节点，其他节点充值选举定时器