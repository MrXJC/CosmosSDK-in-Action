# ABCI
> **ABCI** is an interface that defines the boundary between the replication engine (the blockchain), and the state machine (the application).

应用区块链接口（ABCI）其实就是把共识引擎的逻辑和P2P层分离开来。ComosSDK的模块就是实现了共识引擎的逻辑。这个接口被实现为一个socket协议。

## Message Type

* `BeginBlock` 开始打包区块时候执行的消息
* `DeliveTx` 消息是应用的主要部分。链中的每笔交易都通过这个消息进行传送。应用需要基于当前状态，应用协议，和交易的加密证书上，去验证接收到 `DeliverTx` 消息的每笔交易，。一个经过验证的交易然后需要去更新应用状态 – 比如通过将绑定一个值到键值存储(KVstore)。
* `CheckTx` 消息类似于 `DeliverTx`，但是它仅用于验证交易。Tendermint Core 的内存池首先通过 `CheckTx` 检验一笔交易的有效性，并且只将有效交易中继到其他节点。比如，一个应用可能会检查在交易中不断增长的序列号，如果序列号过时，`CheckTx` 就会返回一个错误。又或者，他们可能使用一个基于容量的系统，该系统需要对每笔交易重新更新容量。
* `EndBlock`  打包区块结束时候执行的消息
* `commit` 消息用于计算当前应用状态的一个加密保证（cryptographic commitment），这个加密保证会被放到下一个区块头。这有一些比较方便的属性。现在，更新状态时的不一致性会被认为是区块链的分支，分支会捕获所有的编程错误。这同样也简化了保障轻节点客户端安全的开发，因为 Merkel-hash 证明可以通过在区块哈希上的检查得到验证，区块链哈希由一个 quorum 签署。

一般我们CosmosSDK 模块 实现的接口就是`CheckTx`,`DeliverTx`,`BeginBlock`,`EndBlock`这四种。

## Staking Transaction Procedure
> ABCI其实就是Tendermint 和 ComosSDK的桥梁（也就是我们要写的模块）。

下面这张图是我自己画的整个comosSDK和tendermint通过ABCI连接的消息传递的架构图（没有画cosmosSDK 返回给tendermint结果的部分）：
![img](../res/pic/ABCI-cosmosSDK-Tendermint.png)

然后我通过讲解某用户发送申请成为一个验证人的交易（我们可以称为`MsgDeclareCandidacy`）流程，来更好的理解我画的图和ABCI接口的牛逼之处。

首先用户通过客户端发送交易信息`MsgDeclareCandidacy`，`MsgDeclareCandidacy`首先会发送到TendermintCore这时候就会调用`checktx`，然后没有问题就放入mempool中，等待被打包。此时验证人节点开始打包 ，会调用`beginblock`，然后正好把`MsgDeclareCandidacy`放入打包的序列里面，当处理`MsgDeclareCandidacy`的时候会调用delivertx，主要是实现业务逻辑，存取kvstore，之后返回结果。打包最后会调用endBlock，做一些收尾工作，比如更新验证人集。最后，`commit`，当然`commit`干什么可以看上文。

然后调用`checktx`,`delivertx`,`beginblock`,`endblock`的流程是相似的，我用`checktx`和上图来进行讲解，首先TendermintCore会调用`checktx`接口来检查信息合法性，通过ABCI协议远程调用ComosSDK中的Baseapp的`checktx`。Baseapp有一个`Router`模块，可以识别这个tx是哪个模块的，然后转发给相应模块的`checktx`触发。（对于`Beginblock` 和 `endblock` ，其实都只在baseapp.go中注册一次，因为这两个接口都是和消息类型无关的，执行的是一个块开始和结束的操作，所以我觉得这两个接口内部需要调用所需模块的功能。）








