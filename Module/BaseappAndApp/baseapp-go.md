# baseapp.go 

> Loc: baseapp/baseapp.go

ABCI应用的基本结构

```go
// The ABCI application
type BaseApp struct {
	// initialized on creation
	Logger     log.Logger
	name       string               // application name from abci.Info
	db         dbm.DB               // common DB backend
	cms        sdk.CommitMultiStore // Main (uncached) state
	router     Router               // handle any kind of message
	codespacer *sdk.Codespacer      // handle module codespacing

	// must be set
	txDecoder   sdk.TxDecoder   // unmarshal []byte into sdk.Tx
	anteHandler sdk.AnteHandler // ante handler for fee and auth

	// may be nil
	initChainer  sdk.InitChainer  // initialize state with validators and state blob
	beginBlocker sdk.BeginBlocker // logic to run before any txs
	endBlocker   sdk.EndBlocker   // logic to run after all txs, and to determine valset changes

	//--------------------
	// Volatile
	// checkState is set on initialization and reset on Commit.
	// deliverState is set in InitChain and BeginBlock and cleared on Commit.
	// See methods setCheckState and setDeliverState.
	// .valUpdates accumulate in DeliverTx and are reset in BeginBlock.
	// QUESTION: should we put valUpdates in the deliverState.ctx?
	checkState   *state           // for CheckTx
	deliverState *state           // for DeliverTx
	valUpdates   []abci.Validator // cached validator changes from DeliverTx
}
```
* `name` 就是应用的名称
* `router` 路由选择正确的handler进行处理
* `anteHandler` 每一笔交易都会执行手续费和验证的handler
* beginBlocker 和 endBlocker就是 对应的abci接口，一个在所有交易之前执行，一个是在交易执行之后执行。
* checkState 和 deliverState表示的是当前处理的交易进入了什么阶段。
* valUpdates 是缓存验证人集的更新，一般在endblocker的时候更新

ABCI 几个典型接口

```go
// Implements ABCI
func (app *BaseApp) DeliverTx(txBytes []byte) (res abci.ResponseDeliverTx) {
	// Decode the Tx.
	var result sdk.Result
	var tx, err = app.txDecoder(txBytes)
	if err != nil {
		result = err.Result()
	} else {
		result = app.runTx(false, txBytes, tx)
	}

	// After-handler hooks.
	if result.IsOK() {
		app.valUpdates = append(app.valUpdates, result.ValidatorUpdates...)
	} else {
		// Even though the Result.Code is not OK, there are still effects,
		// namely fee deductions and sequence incrementing.
	}

	// Tell the blockchain engine (i.e. Tendermint).
	return abci.ResponseDeliverTx{
		Code:      uint32(result.Code),
		Data:      result.Data,
		Log:       result.Log,
		GasWanted: result.GasWanted,
		GasUsed:   result.GasUsed,
		Tags:      result.Tags,
	}
}
```
上面就是一个DeliverTx的grpc调用的接口实现。
消息是通过go-amino序列化过所以他先用`app.txDecoder(txBytes)`解码，再调用`runTx`，
其实大部分交易tx 都是在runTx中处理的，然后返回处理结果。
`app.valUpdates = append(...`添加改变的验证人，最后把`abci.ResponseDeliverTx`返回给tendermint core

这时候大家肯定很好奇runTx里面到底做了什么？
让我继续跟踪吧（因为runtx函数源码很长所以我删去了一部分对理解没有关系的代码）

```go
// txBytes may be nil in some cases, eg. in tests.
// Also, in the future we may support "internal" transactions.
func (app *BaseApp) runTx(isCheckTx bool, txBytes []byte, tx sdk.Tx) (result sdk.Result) {
	// Handle any panics.
	defer func() {...}()
	// Get the Msg.
	var msg = tx.GetMsg()
	...
	// Validate the Msg.
	err := msg.ValidateBasic()
	...
	// Get the context
	var ctx sdk.Context
	if isCheckTx {
		ctx = app.checkState.ctx.WithTxBytes(txBytes)
	} else {
		ctx = app.deliverState.ctx.WithTxBytes(txBytes)
	}
	// Run the ante handler.
	if app.anteHandler != nil {
		newCtx, result, abort := app.anteHandler(ctx, tx)
		...
	}
	// Match route.
	msgType := msg.Type()
	handler := app.router.Route(msgType)
	...
	// Get the correct cache
	var msCache sdk.CacheMultiStore
	if isCheckTx == true {
		// CacheWrap app.checkState.ms in case it fails.
		msCache = app.checkState.CacheMultiStore()
		ctx = ctx.WithMultiStore(msCache)
	} else {
		// CacheWrap app.deliverState.ms in case it fails.
		msCache = app.deliverState.CacheMultiStore()
		ctx = ctx.WithMultiStore(msCache)
	}
	result = handler(ctx, msg)
	// If result was successful, write to app.checkState.ms or app.deliverState.ms
	if result.IsOK() {
		msCache.Write()
	}

	return result
}

```
输入参数`isCheckTx`代表了这个runtx是checktx调用的还是delivertx调用的，因为这两个接口，有些操作和资源需求的是不一样的。但设计的时候delivertx和checktx都会走runtx。
`GetMsg()`获取`msg`，`msg.validateBasic()`验证交易合法性，ctx就相当于处理tx所需要的所有资源。`msg.type()`获取交易消息的类型。
`handler := app.router.Route(msgType)`这个就是根据交易信息类型选择handler，这个Route的具体应用就在这里，根据模块路由消息到对应的handler。`handler(ctx,msg)`开始调用你编写的`handler.go`里面的代码了（其实对应的handler肯定有一个参数会判别这个消息是checkt还是delivertx，因为上面说过他们两走一个，后面源码分析也会看到有一个变量决定了），返回消息处理的结果。所以在app.go对baseapp.go二次封装的时候，必定要要把自己模块的handler注册到router上。

**注意** 大家有没有注意到 `antehandler`,这个是不管是那个模块对应的交易消息都会执行的handler，我猜以后会实现**手续费**啊之类的。

##总结
baseapp.go 定义了一个基本的abci应用的结构和接口。
用户具体的app.go(gaia)都是对baseapp.go的二次封装，添加注册模块。通过route来调用不同的模块的handler，各个模块相互独立。