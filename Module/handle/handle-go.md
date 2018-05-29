# handle.go
> Loc: x/stake/handler.go

handler.go是模块逻辑的核心，所有的模块功能都在handler内部实现。

```go
package stake

import (
	"bytes"

	sdk "github.com/cosmos/cosmos-sdk/types"
	abci "github.com/tendermint/abci/types"
)

//nolint
const (
	GasDeclareCandidacy int64 = 20
	GasEditCandidacy    int64 = 20
	GasDelegate         int64 = 20
	GasUnbond           int64 = 20
)

func NewHandler(k Keeper) sdk.Handler {
	return func(ctx sdk.Context, msg sdk.Msg) sdk.Result {
		// NOTE msg already has validate basic run
		switch msg := msg.(type) {
		case MsgDeclareCandidacy:
			return handleMsgDeclareCandidacy(ctx, msg, k)
		case MsgEditCandidacy:
			return handleMsgEditCandidacy(ctx, msg, k)
		case MsgDelegate:
			return handleMsgDelegate(ctx, msg, k)
		case MsgUnbond:
			return handleMsgUnbond(ctx, msg, k)
		default:
			return sdk.ErrTxDecode("invalid message parse in staking module").Result()
		}
	} 
}
```
NewHandler在内部主要实现了根据模块的msg的具体type来返回不同的handler。stake一共有四种message，所以对应了上面四种类型，然后他返回的也是输入为sdk.Context,sdk.Msg,返回为sdk.Result的函数。下面就是具体handler的实现

```go
// now we just perform action and save
func handleMsgDeclareCandidacy(ctx sdk.Context, msg MsgDeclareCandidacy, k Keeper) sdk.Result {

	// check to see if the pubkey or sender has been registered before
	_, found := k.GetCandidate(ctx, msg.CandidateAddr)
	if found {
		return ErrCandidateExistsAddr(k.codespace).Result()
	}
	if msg.Bond.Denom != k.GetParams(ctx).BondDenom {
		return ErrBadBondingDenom(k.codespace).Result()
	}
	if ctx.IsCheckTx() {
		return sdk.Result{
			GasUsed: GasDeclareCandidacy,
		}
	}

	candidate := NewCandidate(msg.CandidateAddr, msg.PubKey, msg.Description)
	k.setCandidate(ctx, candidate)
	tags := sdk.NewTags("action", []byte("declareCandidacy"), "candidate", msg.CandidateAddr.Bytes(), "moniker", []byte(msg.Description.Moniker), "identity", []byte(msg.Description.Identity))

	// move coins from the msg.Address account to a (self-bond) delegator account
	// the candidate account and global shares are updated within here
	delegateTags, err := delegate(ctx, k, msg.CandidateAddr, msg.Bond, candidate)
	if err != nil {
		return err.Result()
	}
	tags = tags.AppendTags(delegateTags)
	return sdk.Result{
		Tags: tags,
	}
}
```
整个handler函数代码，由`IsCheckTx()`分隔，当checktx调用的时候，返回`true`，当是deliverTx调用的时刻，返回`false`。换句话说，`IsCheckTx()`上面部分是checkTx处理的逻辑，用来检查交易的合法性，然后deliverTx处理的逻辑是包含checkTx的，还多加了具体模块的应用逻辑
，对KVStore进行读写操作。最后就是返回`sdk.Result`。注意每个handler末尾都有Tag，记录handler处理的信息返回给tendermintCore。`sdk.NewTags`的输入是`string`类型然后接`[]bytes`类型，成对出现的变长参数列表。
