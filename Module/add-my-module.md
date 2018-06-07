##添加自己的mymodule到app.go
> Loc:github.com/mrxjc/

下面是我具体添加mymodule的代码片段，具体细节见注释。

```go
// Extended ABCI application
type GaiaApp struct {
	*bam.BaseApp
	cdc *wire.Codec

	// keys to access the substores
	...
	// 1.添加自己的key
	keyMymodule *sdk.KVStoreKey

	// Manage getting and setting accounts
	...
	// 2.添加自己的keeper
	mymoduleKeeper mymodule.Keeper
}

func NewGaiaApp(logger log.Logger, db dbm.DB) *GaiaApp {
	cdc := MakeCodec()

	// create your application object
	var app = &GaiaApp{
	...
	// 3.初始化自己的key
		keyMymodule: sdk.NewKVStoreKey("mymodule"),
	}

	// define the accountMapper
    ...
    
	// add handlers
	...
	// 4.初始化自己的keeper
	app.mymoduleKeeper = mymodule.NewKeeper(app.cdc, app.keyMymodule, app.coinKeeper,app.RegisterCodespace(mymodule.DefaultCodespace))
	// register message routes
   ...
   // 5.添加自己的module的handler
	app.Router().AddRoute("mymodule",mymodule.NewHandler(app.mymoduleKeeper))

	// initialize BaseApp
	// 6.初始化区块链的时刻调用的handler
	app.SetInitChainer(app.initChainer)
	// 7.设置每次beginblock时刻调用的handler
	app.SetBeginBlocker(gov.NewBeginBlocker(app.govKeeper))
	// 8.设置每次endblock时刻调用的handler
	app.SetEndBlocker(stake.NewEndBlocker(app.stakeKeeper))
	// 9.添加自己的key到StoresIAVLtree中
	app.MountStoresIAVL(app.keyMain, app.keyAccount, app.keyIBC, app.keyStake, app.keyGov,app.keyMymodule)
	// 10.设置每次消息都会调用的handler
	app.SetAnteHandler(auth.NewAnteHandler(app.accountMapper, stake.FeeHandler))
    ...
	return app
}

// custom tx codec
func MakeCodec() *wire.Codec {
	var cdc = wire.NewCodec()
    ...
    // 11.注册自己的Wire
	mymodule.RegisterWire(cdc)
	return cdc
}
```
