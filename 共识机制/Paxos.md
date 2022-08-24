Paxos算法于1990年由LeslieLamport在论文ThePart-timeParliament中首次提出，其实现了一种最大化保证分布式系统一致性的机制。但由于最初发布的论文中描述比较晦涩难懂，2001年，作者LeslieLamport专门发表论文PaxosMadeSimple进行进一步的解释说明。

Paxos算法包括三种角色，即 #提案者 （Proposer）、 #受理者 （Acceptor）和 #同步者 （Learner）。提案者需要先对某 #提案 （Proposal，由编号n进行区分）进行投票，得到大多数受理者的支持后，才将该提案再发送至所同步者进行确认。

Paxos并不保证系统总处于强一致的状态，但由于每次提案共识均需要至少超过一半的节点同意，最终整个系统会达到一致的状态，即保证最终一致性。

目前Google的Spanner、Chubby等系统中均应用了Paxos算法。但Paxos算法由于其原理细节描述较少以及很高的工程复杂度，实际应用中常以Paxos算法为基础，对其算法流程进行优化更新，得到Paxos的改进算法，以更适用于工程应用，其中最经典的即[[Raft]]共识算法。

### 算法流程
算法流程可由如下三阶段表示（其中决议过程为前两阶段）
![[Paxos算法流程示意图.png|400]]
#### Prepare阶段
Proposer对某提案生成全局唯一的编号n并广播Prepare请求至Acceptors，由Acceptor进行投票。
当Acceptor收到一个Proposal后，对比其编号n与本地记录的已经投票过的最大提案编号max，若n大于max则对该提案投赞成票，回复Proposer投票结果，同时承诺不再接受任何编号小于n的提案。
#### Accept阶段
如果Proposer收到超过一半的Acceptor对提案n投赞成票，则Proposer广播Accept请求至这些Acceptors，Accept请求内容为一个键值对\[Key，Value\]，其中Key为提案编号n，Value为决议值，代表提案本身内容。Acceptor收到编号为n的提案Accept请求后，检查其是否对高于n的提案投过Prepare赞成票，若没有则接受该请求，并回复Proposer。
#### 同步阶段
Proposer收到多数受理者的Accept消息后，表明本次提案决议成功，并将决议发给所有Learner节点同步。
