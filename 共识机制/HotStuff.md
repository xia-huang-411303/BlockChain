HotStuff协议由Abraham、Gueta等人提出，是一种在[[PBFT]]等共识算法的基础上实现的三阶段投票的BFT共识算法。
该协议满足共识算法安全性(Safety)、活性(Liveness)和响应性(Responsiveness)，同时可以实现线性的消息复杂度，即使在Leader转换过程中依然可保持为线性。

PBFT是最早的可实用的拜占庭容错共识算法，但由于其在 #视图转换 时需要大量的消息转发，导致每当共识集群需要切换Leader时消息复杂度过高，在实际项目中很难承受。

### HotStuff共识过程
HotStuff是基于 #View (视图，配置状态)的共识协议，一个View表示一个 #共识单元 ，使用ViewNumber表示，共识过程由一个接一个的View组成。其中View相当于PBFT中 #视图 的概念。
在一个View中，存在一个Leader来主导共识协议，共识流程分为Prepare、Pre-Commit和Commit三个阶段，经过三阶段投票达成共识后切换到下一个View继续进行共识。
如遇到异常情况，导致某个View超时未能达成共识，不会影响切换下个View继续进行共识流程。
共识过程中通过 #QC (QuorumCertificates)表示某区块的投票证明，其内容可包含被$(n-f)$个节点签名确认的数据集合及ViewNumber。
HotStuff共识过程的三阶段如图所示。图即ViewNumber为N的View共识过程。
![[HotStuff三阶段共识流程.png|500]]
#### Prepare阶段
每个View开始时，当前View的Leader(类似于PBFT中 #Primary 节点角色)收集由(n－f)个副本节点发送的New-View消息，Leader收到消息后，选择ViewNumber最高的分支，基于该分支创建新的共识请求，并广播给各副本节点。各副本节点一旦收到当前Leader对应的提案，校验提案合法后，则向Leader回复一个经过自己私钥签名的Prepare-Vote。
#### PreCommit阶段
Leader发出提案消息后，等待(n－f)个节点对该提案的投票签名，集齐投票签名后会将这些签名组合成一个新的签名，表明有(n－f)个节点对该提案投过票。然后组装PreCommit消息广播至各副本节点，副本节点收到消息后验证签名，验证通过后表示第一轮Prepare投票成功。最后用自身私钥对PreCommit消息签名投票并回复给Leader。
#### Commit阶段
Commit阶段与PreCommit阶段类似，也是先由Leader先收集(n－f)个对PreCommit消息的投票签名，组装成Commit消息并广播给各副本节点。然后各副本节点验证通过消息签名后，表明第二轮Commit投票成功。最后用自身私钥对Commit消息签名投票并回复给Leader。
#### Decide阶段
当Leader节点收到了(n－f)个对Commit消息的投票签名，则表示第三轮Commit阶段投票成功，然后将Decide消息广播至各副本节点。副本收到Decide消息后，认为当前提案投票成功，最后执行相应交易，并将ViewNumber加1，开始新一轮View的共识流程。

在上述阶段中，副本节点侧有一个定时器，若在一个View中定时器超时，则将递增ViewNumber，开始新的View。在PBFT算法中，投票消息是节点间相互广播，各个节点都要做投票收集工作，每轮投票，各个副本节点都需要至少验证(n－f)个签名。HotStuff中副本节点利用各自的私钥对信息签名，仅由Leader节点收集签名，不需广播至所有节点，Leader节点收到足够数量的签名消息后，通过门限签名算法将各副本节点的签名信息联合成一个新的签名信息，再在下一个阶段作为证据发给所有副本节点。
此种方法的每一轮投票，每个副本节点只需要验证一次签名。因此，HotStuff通过引入门限签名方案避免了各副本节点之间广播消息，把复杂度从PBFT的$O(n^2)$降低到$O(n)$。同时，HotStuff将视图转换融入算法正常流程，极大地简化了视图转换的复杂度。
由上述流程可看出，在基本HotStuff三阶段共识流程中，每一阶段动作均相似，即先由Leader向副本节点发出消息然后收集投票。

## 链式HotStuff

![[链式HotStuff示意图.png|500]]
MaofanYin等提出链式HotStuff协议(Chained-HotStuff)(见上图)，并对协议的安全性进行了严格证明，该协议将HotStuff各阶段做并行化流水线处理以简化协议，同时提升协议性能。
链式HotStuff具体来说，即在Prepare阶段的投票由当前View对应的Leader1收集，集齐后将Prepare消息发送至下一个View的Leader2，Leader2基于Prepare消息开始新的Prepare阶段，此为View2的Prepare阶段，同时是View1的PreCommit阶段。以此类推，Leader2产生新的Prepare消息然后发送给Leader3开始View3，View3开始Prepare阶段，同时是View2的PreCommit阶段，View1的Commit阶段。
在链式HotStuff中，一个区块的确认是对其直接父区块的确认，称之为1-chain，同理，一个区块b后面连续产生了k个有效区块，则称这段分支是对区块b的k-chain。每当一个新的区块生成后，节点都会检查是否对前述区块形成1-chain、2-chain、3-chain。1-chain表明区块Prepare阶段投票成功；2-chain表明区块链PreCommit阶段投票成功；3-chain表明区块Commit阶段投票成功，即可以成功确认区块，然后执行相关交易。因此只要一个区块后面产生了3个连续的区块，即可表明该区块已经通过了三轮投票确认，其中的交易便可以确认。

## 总结
传统 #BFT 共识机制一般是进行Prepare(定序)和Commit两轮共识，然后落盘，即先达成共识，然后上链。而链式HotStuff通过“3-chain”原则，结合聚合签名算法，先上链再共识，可省去“定序”阶段。抛开传统两轮共识思想，在链式HotStuff中，每一轮流程均相同，即Leader负责出块和收集签名进行聚合，其他节点负责对Leader出的块进行签名，只要Leader收到了(n－f)个签名，就可以出一个区块，该区块后面有3个连续的区块就相当于已达成共识。

HotStuff算法还有一个创新是在算法实现中将安全性和活性进行了解耦，两者可以独立设计，算法的安全性有严格的数学证明，用户不必担心，可根据实际系统灵活设计算法活性部分，也称为算法的Pacemaker模块。

HotStuff自提出以来，其以同时具备共识协议安全性、活性和响应性，以及更低的消息复杂度及链式流水线优化结构，使其迅速成为BFT共识家族重要的一分子。目前Facebook推出的加密货币项目Libra中使用的共识算法LibraBFT是在基于链式HotStuff的基础上进行了更新优化，其确认规则遵从链式HotStuff的3-chain的确认规则，并根据Libra系统提供了一个详细的算法活性设计和Pacemaker实现。
LibraBFT协议同时提供了对共识节点投票权力的重配置机制和节点的奖惩机制，以提供安全高效的BFT协议。