# Tendermint
> [Tendermint](http://tendermint.readthedocs.io/en/master/)
是一个PoS，基于权益证明的、拜占庭容错的共识引擎。

## 特点
* 第一，它是BFT拜占庭容错的，可以最多容忍三分之一的拜占庭节点。
* 第二,就是快速最终性得到共识后形成的最新区块就是最终区块，这种最终性不像PoW是基于概率性的。
* 第三,就是它的共识效率非常高的，每秒钟可以确认上千笔交易。

Tendermint实现了网络层和共识层的功能，而让开发人员通过ABCI接口实现应用层的逻辑。这样的设计让Tendermint成为了一个通用的共识引擎，你可以在上面实现各种定制化的功能。

## Detail
Tendermint doc [http://tendermint.readthedocs.io/en/master/](http://tendermint.readthedocs.io/en/master/)