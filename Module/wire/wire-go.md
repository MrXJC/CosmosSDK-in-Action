# wire.go
> Loc: x/stake/wire.go

```go
package stake

import (
	"github.com/cosmos/cosmos-sdk/wire"
)

// Register concrete types on wire codec
func RegisterWire(cdc *wire.Codec) {
	cdc.RegisterConcrete(MsgDeclareCandidacy{}, "cosmos-sdk/MsgDeclareCandidacy", nil)
	cdc.RegisterConcrete(MsgEditCandidacy{}, "cosmos-sdk/MsgEditCandidacy", nil)
	cdc.RegisterConcrete(MsgDelegate{}, "cosmos-sdk/MsgDelegate", nil)
	cdc.RegisterConcrete(MsgUnbond{}, "cosmos-sdk/MsgUnbond", nil)
}
```
注册stake的Msg的消息实例，供go-amino来进行序列化，如果你想要添加自己mymodule的msg的话，见wire.go-template.

