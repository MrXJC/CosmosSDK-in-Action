# keeper.go
>Loc: x/stake/keeper.go

每个模块都有自己的KVstore，用来存储模块执行的需要保存的结果（对bank，就是账户余额...,对于stake 就是pool，还有validator，delegator的一些信息），keeper就是访问和修改KVStore入口。KVStore的状态，决定了程序的运行状态，如果一个节点的KVStore的apphash与其他节点不同，他就无法继续运行。

```go
package stake

import (
	"bytes"

	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/wire"
	"github.com/cosmos/cosmos-sdk/x/bank"
	abci "github.com/tendermint/abci/types"
)

// keeper of the staking store
type Keeper struct {
	storeKey   sdk.StoreKey
	cdc        *wire.Codec
	coinKeeper bank.Keeper

	// caches
	pool   Pool
	params Params

	// codespace
	codespace sdk.CodespaceType
}

func NewKeeper(cdc *wire.Codec, key sdk.StoreKey, ck bank.Keeper, codespace sdk.CodespaceType) Keeper {
	keeper := Keeper{
		storeKey:   key,
		cdc:        cdc,
		coinKeeper: ck,
		codespace:  codespace,
	}
	return keeper
}
```
stake中的keeper的struct中你会发现有coinKeeper，这个是对账户KVStore的存取，所以一个专属于模块的keeper中也可以调用其他模块的keeper来进行KVStore的访问。stake模块因为需要对用户账户的KVStore进行存取操作，所以在stake中keeper引用了coinkeeper。当然你需要其他的模块的KVStore，你也可以引入。

* `storeKey` 就是你kvstore的name，在app.go中注册的字符串。
* `Pool`,`Params`就是stake模块kvstore需要存取的数据结构

```go
// get the current in-block validator operation counter
func (k Keeper) getCounter(ctx sdk.Context) int16 {
	store := ctx.KVStore(k.storeKey)
	b := store.Get(CounterKey)
	if b == nil {
		return 0
	}
	var counter int16
	err := k.cdc.UnmarshalBinary(b, &counter)
	if err != nil {
		panic(err)
	}
	return counter
}

// set the current in-block validator operation counter
func (k Keeper) setCounter(ctx sdk.Context, counter int16) {
	store := ctx.KVStore(k.storeKey)
	bz, err := k.cdc.MarshalBinary(counter)
	if err != nil {
		panic(err)
	}
	store.Set(CounterKey, bz)
}
```
`getCounter`和`setCounter`就是典型的存取。ctx 就是context，保存了模块运行所需要的所有资源，比如kvstore就在里面。根据`storeKey`来获取对应模块的`KVStore`。

* `getCounter` 通过ctx.KVStore获取stake的store。通过store.Get(counter），获取counter(CounterKey = []byte{0x08})对应的值，这个值也是字节数组。然后通过`k.cdc.UnmarshalBinary`函数获取真实的值，赋值给counter，最后返回counter
* `setCounter` 与get操作相反，传入你需要写的值counter，一样获取store，然后用`k.cdc.MarshalBinary`编码，最后调用store.Set,把bz的值写入CounterKey对应的存储里。