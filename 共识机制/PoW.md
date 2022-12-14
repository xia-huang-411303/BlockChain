工作量证明是一种应对拒绝服务攻击和其他服务滥用的经济对策。它要求发起者进行一定量的运算，也就意味着需要消耗计算机一定的时间。

这个概念由Cynthia Dwork和Moni Naor于1993年在学术论文中首次提出。而“工作量证明”这个名词，则是1999年在Markus Jakobsson和Ari Juels的文章中才被真正提出。
在比特币之前，哈希现金被用于垃圾邮件的过滤，也被微软用于Hotmail/Exchange/Outlook等产品中。

工作量证明系统的主要特征是客户端需要做一定难度的工作得出一个结果，验证方却很容易通过结果来检查出客户端是不是做了相应的工作。这种方案的一个核心特征是不对称性：工作对于请求方是困难的，对于验证方则是简单的。具体来讲，每个可出块节点通过不断猜测一个数值（Nonce），使得该数值可以使所出块中所包含的交易内容的哈希值满足一定条件。由于哈希问题在目前的计算模型下是一个不可逆的问题，除了反复猜测数值，进行计算验证外，还没有有效的方法能够逆推计算出符合条件的Nonce值。且比特币系统可以通过调整计算出的哈希值所需要满足的条件来控制计算出新区块的难度，从而调整生成一个新区块所需的时间的期望值。Nonce值计算的难度保证了在一定的时间内，整个比特币系统中只能出现少数合法提案。另一方面，在节点生成一个合法提案后，会将提案在网络中进行广播，收到的用户在对该提案进行验证后，会基于它所认为的最长链的基础上继续生成下一个分叉。这种机制保证了系统中虽然可能会出现分叉，但最终会有一条链成为最长的链，被绝大多数节点所共识。

然而，由于比特币采用SHA-256算法，挖矿速度与机器算力成正比，这就催生了专门的挖矿专用集成电路，即矿机。矿机的挖矿效率相比于普通的GPU高数个数量级，带来的影响是算力越发集中于专用矿场，使得普通用户难以入场，降低了区块链的去中心化程度。
针对这个问题，莱特币采用了一种“内存难题算法”——Scrypt作为其挖矿算法，其求解速度主要取决于计算机内存大小，这大大降低了大规模矿场在莱特币中的优势，使得去中心化这一特性得以保证。
当前，以太坊所采用的PoW算法——Ethash也与Scrypt类似，是一种“内存难题算法”，其所要解决的主要问题也是专用矿机相对于普通PC在挖矿上展现出来的巨大优势。
