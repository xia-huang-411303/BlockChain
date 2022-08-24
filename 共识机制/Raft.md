Raft算法即以Paxos为基础，最初由斯坦福大学的DiegoOngaro和JohnOusterhout于2013年在发表的[论文](https://web.stanford.edu/~ouster/cgi-bin/papers/raft-atc14)中提出，在DiegoOngaro的博士论文Consensus：BridgingTheoryandPractice中，对该算法的细节进行了详尽的描述。

该算法对Paxos进行了简化设计和流程优化，比Paxos更易于理解和实现，在实际应用场景中更加实用。Raft算法的核心思想是集群中各节点从相同的初始状态开始，以相同的顺序执行各个操作，最终使集群达成一致的状态。

### 角色划分
Raft算法将节点划分为三种角色，即 #领导者 （Leader）、 #跟随者 （Follower）、 #候选者 （Candidate），三种角色之间可以相互转换。
Raft算法中节点三种角色状态的转换如图。
![[Raft算法角色的状态转换.png|400]]
#Leader ：集群中有且只有一个Leader，负责接收客户端的请求、管理日志复制以及与Follower保持心跳（Heartbeat）。
#Follower ：节点启动时均初始化为Follower状态，响应Leader的日志复制请求和Candidate的投票请求。
#Candidate ：Candidate是处于Follower和Leader之间的中间状态。Follower超时后发起Leader选举前需要先转换至Candidate状态，Candidate接收到超过一半以上节点的赞成票后才转换为Leader。

Raft算法中另一个重要的概念是 #任期 （ #Term ）。Term是一个连续递增的编号，每一轮选举是一个Term。在一个任期内可以没有Leader，若存在Leader，则有且仅有一个Leader。

### 共识流程
算法采用日志方式进行同步，通过选举一个Leader进行决策来简化后续流程。Leader决定日志的提交顺序，日志只能由Leader向Follower进行单向复制，以此保证集群中各节点最终的日志顺序一致。
Raft算法共识流程主要分为两个阶段，即Leader选举阶段和日志复制阶段。
#### Leader选举
当Follower超时后增加Term转换为Candidate，向其他节点发送投票请求，等待下述三种情况发生：
1) 赢得选举，即Candidate获得了当前集群超过半数以上节点的赞成票，当选为当前Term的Leader；
2) 输掉选举，即已有另一个节点成了Leader，并收到其发来的心跳消息或日志复制请求，成为Follower；
3) 没有输赢，即本次选举超时结束时集群中仍没有Leader产生，则Candidate继续递增任期号，再发起一次新的选举。
#### 日志复制
Leader节点接收客户端的请求，更新日志条目，向集群中其他节点发送添加日志请求，当集群中超过半数节点成功同步日志后，表示该条日志已成功复制，Leader节点回复客户端。

以上为Raft算法的流程概述，可参考[The Secret Lives of Data](http://thesecretlivesofdata.com/)中的动画更形象地理解Raft算法流程。Raft算法中还有一些其他的特性如集群成员变更、日志压缩等，也可在Ongaro作者博士论文中进行更深入的了解。
目前Raft算法已经在多种主流语言中实现，基于Raft算法的应用也十分广泛，如由Raft保证数据一致性的分布式协调服务etcd和Consul，分布式KV系统TiKV等。
Raft算法目前也已经在各区块链项目中均有支持或实现，如HyperLedgerFabric、XuperChain、蚂蚁链等项目。