假定在一笔交易中，Bob给Alice支付了0.15BTC。由于比特币并没有传统的“账户”概念，用户通过其地址（即其公钥）来标记，因此这笔交易中仅写明了Alice公钥的哈希值。然而，为了限定只有Alice才能够花费这笔交易对应的UTXO，Bob会在这笔0.15BTC的交易中创建一个输出脚本：
`OP_DUP OP_HASH160 <AlicePublicKeyHash> OP_EQUAL OP_CHECKSIG`
这个脚本即表示对于输出交易的解锁条件，即需要提供一个签名和一个公钥。而有效的签名需要用户的私钥生成，因此仅有Alice能够创建出能够通过该脚本验证的签名。Alice在需要花费该交易中的0.15BTC的UTXO时，需要提供Bob生成的锁定脚本所对应的解锁脚本：
`<AliceSignature><AlicePublicKey>`
将解锁脚本和锁定脚本进行组合，获得如下的组合脚本：
`<AliceSignature><AlicePublicKey> OP_DUP OP_HASH160 <AlicePublicKeyHash> OP_EQUAL OP_CHECKSIG` 
该组合脚本的执行过程示意图如图所示。
![[示例组合脚本执行过程1.png]]
![[示例组合脚本执行过程2.png]]
具体地，执行指针从组合脚本的头部开始执行，
首先遇到<Signature>及<PubKey>两个操作数，则按顺序将其压入堆栈；
执行指针继续向后移动，遇到一元操作符“DUP”，该操作符的作用为复制栈顶元素并将其压入栈顶，执行完成后堆栈中现有元素按从栈顶到栈底的顺序排列有<PubKey>、<PubKey>、<Signature>三个；
执行指针继续后移，遇到一元操作符“HASH160”，该操作符的作用为计算栈顶元素的哈希值，并将计算结果压入栈顶；
计算完后，堆栈中现有元素的排列变成了<PubKeyHash>、<PubKey>、<Signature>；
执行指针继续后移后遇到操作数<PubKeyHash>，直接压入栈顶，堆栈中元素变为<PubKeyHash>、<PubKeyHash>、<PubKey>、<Signature>；
执行指针指向的下一个操作符为二元操作符“EQUALVERIFY”，该操作符的作用为对两个操作数进行判等，若判等通过，则将两操作数移除，并继续执行；
EQUALVERIFY成功执行完毕后，堆栈中元素变为<PubKey>、<Signature>，执行指针继续右移；
执行指针最终指向二元操作符“CHECKSIG”，该操作符会对一组公钥和签名进行检查，确认签名是由公钥对应的私钥生成的；
执行完毕后，会将对应的执行结果压入栈中。

可以看到，只有当解锁脚本与锁定脚本的设定条件相匹配时，执行组合脚本时才会显示结果为真（Ture）。即只有当解锁脚本提供了Alice的有效签名，交易执行结果才会被通过（结果为真）。
