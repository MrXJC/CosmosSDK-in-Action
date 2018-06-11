# Architecture

我们可以先参考一下gaia（测试网）和 stake模块的一些具体结构l来研究如何在gaia测试网基础上创建并 添加自己的Cosmos Module。

```
CosmosSDK
├── baseapp
│   └── baseapp.go
├── cmd
│   └── gaia
│       ├── app
│       │    └── app.go
│       └── cmd
│           ├── gaiacli
│           │   └── main.go
│           └── gaiad
│               └── main.gp 
├── types
└── x
    ├── other-module
    └── stake
        ├── client
        │   ├── cli
        │   │   ├── flags.go
        │   │   ├── query.go
        │   │   └── tx.go
        │   └── rest
        │        └── query.go
        ├── errors.go
        ├── handler.go
        ├── keeper.go
        ├── msg.go
        ├── types.go
        ├── wire.go
        └── other go file....       
```
## baseapp
定义了基本的ABCI的应用，具体实现了`beginblock`,`checktx`,`delivertx`,`endblock`....(tendermint完成消息的广播和共识，验证的细节都交给ABCI，其实就是一个GRPC的调用，我其实觉得很牛逼，完全让两者解耦合)。然后baseapp里面有一个route，这个模块很重要其实就是根据消息`msg`的类型来调用具体模块的`beginblock`,`checktx`,`delivertx`,`endblock`....下面会讲具体baseapp加载stake模块的概要。

## cmd
这个文件下最主要的是app.go,他主要就是基于baseapp.go,二次封装，把自己写的模块的handler，keeper，kvstore's key 注册进来。

* gaiacli 就是客户端用来发送给交易，或者说发送一些触发事件的消息。
* gaiad   就是服务端，相当于测试网的全节点。

gaiacli gaiad 下的main.go 都是调用app.go 里面的方法。
## types
里面定义了一些各个模块之间通用的结构和接口，例如Address，Account，Coins，Handler，StdSignature....随后我结合stake源码会在附录添加一些我自己觉得比较常用的结构与接口的细节。
## X/stake
1. **types.go** 定义了一些此模块需要的结构，对于stake需要定义validator，delegator（除此之外他还特地新建了pool.go,来定义抵押池，实际上也可以放入types）
2. **msg.go** 定义了此模块的接受的客户端交易信息的数据类型。对于stake来说就只有四种交易信息： 
  * 申请成为验证信息
  * 修改验证节点信息
  * 向验证节点抵押贷币
  * 从验证节点解绑贷币
3. **keeper.go** 主要是在KVstore中对需要保存的模块程序状态数据进行存取。
4. **handler.go** 根据msg的类型调用相应的keeper实现代码逻辑，数据存取，返回信息给tendermint，一般就是实现abci接口的 checktx 和 delivertx。对于stake来说，也就是四种方法（在msg.go中定义）。
5. **wire.go**  go-amino序列化的注册（为tendermint与cosmosSDK之间消息传递）
6. **error.go** 定义一些关于stake的错误类型 

## Example/mymodule
源码分析CosmosSDK的stake模块，然后再来从零实现自己的mymodule。大概流程是 分析完stake的msg.go,再构建mymodule的msg.go.如果你想先体验一下mymodule的功能，你可以提前看Example的章节。