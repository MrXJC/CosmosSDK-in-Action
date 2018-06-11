# msg.go
> Loc:x/stake/msg.go

> 简单分析源码，然后在Example中实现了mymodule模块
 
```go
package stake

import (
	"encoding/json"

	sdk "github.com/cosmos/cosmos-sdk/types"
	"github.com/cosmos/cosmos-sdk/wire"
	crypto "github.com/tendermint/go-crypto"
)

// name to idetify transaction types
const MsgType = "stake"

// XXX remove: think it makes more sense belonging with the Params so we can
// initialize at genesis - to allow for the same tests we should should make
// the ValidateBasic() function a return from an initializable function
// ValidateBasic(bondDenom string) function
const StakingToken = "steak"

//Verify interface at compile time
var _, _, _, _ sdk.Msg = &MsgDeclareCandidacy{}, &MsgEditCandidacy{}, &MsgDelegate{}, &MsgUnbond{}

var msgCdc = wire.NewCodec()

func init() {
	wire.RegisterCrypto(msgCdc)
}
```
* `MsgType`是消息模块的名字，唯一确定在消息来自哪个模块，一个模块可以对应很多消息，stake的消息如下：
    * MsgDeclareCandidacy
    * MsgEditCandidacy
    * MsgDelegate
    * MsgUnbond
    
* `StakingToken`就是用于抵押的token的名字

```go
//______________________________________________________________________

// MsgDeclareCandidacy - struct for unbonding transactions
type MsgDeclareCandidacy struct {
	Description
	CandidateAddr sdk.Address   `json:"address"`
	PubKey        crypto.PubKey `json:"pubkey"`
	Bond          sdk.Coin      `json:"bond"`
}

func NewMsgDeclareCandidacy(candidateAddr sdk.Address, pubkey crypto.PubKey,
	bond sdk.Coin, description Description) MsgDeclareCandidacy {
	return MsgDeclareCandidacy{
		Description:   description,
		CandidateAddr: candidateAddr,
		PubKey:        pubkey,
		Bond:          bond,
	}
}

//nolint
func (msg MsgDeclareCandidacy) Type() string              { return MsgType } //TODO update "stake/declarecandidacy"
func (msg MsgDeclareCandidacy) GetSigners() []sdk.Address { return []sdk.Address{msg.CandidateAddr} }

// get the bytes for the message signer to sign on
func (msg MsgDeclareCandidacy) GetSignBytes() []byte {
	return msgCdc.MustMarshalBinary(msg)
}

// quick validity check
func (msg MsgDeclareCandidacy) ValidateBasic() sdk.Error {
	if msg.CandidateAddr == nil {
		return ErrCandidateEmpty(DefaultCodespace)
	}
	if msg.Bond.Denom != StakingToken {
		return ErrBadBondingDenom(DefaultCodespace)
	}
	if msg.Bond.Amount <= 0 {
		return ErrBadBondingAmount(DefaultCodespace)
	}
	empty := Description{}
	if msg.Description == empty {
		return newError(DefaultCodespace, CodeInvalidInput, "description must be included")
	}
	return nil
}
```
上面的代码片段，就是定义一个典型消息的代码。以`MsgDeclareCandidacy`为例子介绍：

1. 定义消息的struct
2. newMsg() 初始化所有的消息信息
3. 定义`Type()`返回模块的MsgType
4. `GetSigners()` 获取交易的发起者，就是签名的人
5. `GetSignBytes()` 把整个交易序列化成字节数组，然后用来被签名
6. `ValidateBasic()` 基本的对交易信息逻辑的检测，看看有没有问题，stake中就检测了bond的数量是不是大于0，token的名称是不是`steak`

就上面6部分组成一个msg的定义

```go
//______________________________________________________________________

// MsgEditCandidacy - struct for editing a candidate
type MsgEditCandidacy struct {
	Description
	CandidateAddr sdk.Address `json:"address"`
}
...
//______________________________________________________________________

// MsgDelegate - struct for bonding transactions
type MsgDelegate struct {
	DelegatorAddr sdk.Address `json:"address"`
	CandidateAddr sdk.Address `json:"address"`
	Bond          sdk.Coin    `json:"bond"`
}
...
//______________________________________________________________________

// MsgUnbond - struct for unbonding transactions
type MsgUnbond struct {
	DelegatorAddr sdk.Address `json:"address"`
	CandidateAddr sdk.Address `json:"address"`
	Shares        string      `json:"shares"`
}
...
```

上面的代码片段就是其他的stake的`Msg`的结构，当然他们也有对应的`Type()`,`GetSigners()`,`GetSignBytes()`,`ValidateBasic()`

下一页就具体写一个mymodule的msg.go模版。
