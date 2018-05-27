# app.go
> Loc:cmd/gaia/app/app.go

具体gaiaApp对baseapp的二次封装结构

```go
// Extended ABCI application
type GaiaApp struct {
	*bam.BaseApp
	cdc *wire.Codec

	// keys to access the substores
	keyMain    *sdk.KVStoreKey
	keyAccount *sdk.KVStoreKey
	keyIBC     *sdk.KVStoreKey
	keyStake   *sdk.KVStoreKey

	// Manage getting and setting accounts
	accountMapper sdk.AccountMapper
	coinKeeper    bank.Keeper
	ibcMapper     ibc.Mapper
	stakeKeeper   stake.Keeper
}
```

* `*bam.BaseApp` 这个就是baseapp的基本结构，在他基础上二次开发
* `keyStake   *sdk.KVStoreKey` KVstore的键类型，这是stake的键，如果你想添加自己的module，你可以添加`keyMyModule *sdk.KVStoreKey`
* `stakeKeeper   stake.Keeper` stake.keeper结构(在x/stake/keeper.go里)用来访问和设置kvstore，如果你想要添加自己的module，你可以添加`mymoduleKeeper mymodule.Keeper`



下面就是GaiaApp的构造，完整代码我拆分开来，讲解起来更方便

```go
func NewGaiaApp(logger log.Logger, db dbm.DB) *GaiaApp {
	cdc := MakeCodec()

	// create your application object
	var app = &GaiaApp{
		BaseApp:    bam.NewBaseApp(appName, cdc, logger, db),
		cdc:        cdc,
		keyMain:    sdk.NewKVStoreKey("main"),
		keyAccount: sdk.NewKVStoreKey("acc"),
		keyIBC:     sdk.NewKVStoreKey("ibc"),
		keyStake:   sdk.NewKVStoreKey("stake"),
	}
```
常见应用的对象的时候，如果你想加入你自己的mymodule,你需要像stake模块一样（`keyStake:   sdk.NewKVStoreKey("stake")`）添加自己的`keyMymodule:   sdk.NewKVStoreKey("mymodule")`这个key后面会在app.MountStoresIAVL中注册这个名字的stores，每个模块都有自己的名字的kvstore，虽然现在他们其实是公用一个kvstore，他的根是`'main'`,例如stake模块的kvstore是`'main':{'stake':{...},'mymodule':{...},...}`


```go
	// add handlers
	app.coinKeeper = bank.NewKeeper(app.accountMapper)
	app.ibcMapper = ibc.NewMapper(app.cdc, app.keyIBC, app.RegisterCodespace(ibc.DefaultCodespace))
	app.stakeKeeper = stake.NewKeeper(app.cdc, app.keyStake, app.coinKeeper, app.RegisterCodespace(stake.DefaultCodespace))
```
上面代码就是注册自己模块的keeper，每个keeper构造的是时候如果你想访问其他模块的kvstore也可以穿入其他模块的keeper，例如stakeKeeper 就传入了app.coinKeeper.
然后我可以模仿stake 添加自己的模块代码如下：

```go
app.mymoduleKeeper = mymodule.NewKeeper(app.cdc, app.mymodule,app.coinKeeper,app.stakeKeeper,app.RegisterCodespace(mymodule.DefaultCodespace))
```
Keeper在`keeper.go`中定义。

```go
	// register message routes
	app.Router().
		AddRoute("bank", bank.NewHandler(app.coinKeeper)).
		AddRoute("ibc", ibc.NewHandler(app.ibcMapper, app.coinKeeper)).
		AddRoute("stake", stake.NewHandler(app.stakeKeeper))
```
接下来上面的代码就是注册自己的handler，模仿stake在`app.Router()...`后面添加`.AddRoute（"mymodule"，mymodule.NewHandler(app.mymoduleKeeper)）`这样就成功注册了自己模块的handler.这个在`handler.go`里面声明定义。

```go
	// initialize BaseApp
	app.SetInitChainer(app.initChainer)
	app.SetEndBlocker(stake.NewEndBlocker(app.stakeKeeper))
	app.MountStoresIAVL(app.keyMain, app.keyAccount, app.keyIBC, app.keyStake)
	app.SetAnteHandler(auth.NewAnteHandler(app.accountMapper, stake.FeeHandler))
	err := app.LoadLatestVersion(app.keyMain)
	if err != nil {
		cmn.Exit(err.Error())
	}

	return app
}
```
app.SetEndBlocker注册endblock的handler,上面代码接受的是stake的endblocker，当然你可以定义自己的endblocker模块，但是要注意一个baseapp就一个endblocker接口，所以只能注册一个handler，并不像checktx和delivertx可以通过router来路由选择对应的handler处理。
下面AnteHandler同理
最后app.MountStoresIAVL(中加入 app.keymymodule....)


```go
// custom tx codec
func MakeCodec() *wire.Codec {
	var cdc = wire.NewCodec()
	ibc.RegisterWire(cdc)
	bank.RegisterWire(cdc)
	stake.RegisterWire(cdc)
	auth.RegisterWire(cdc)
	sdk.RegisterWire(cdc)
	wire.RegisterCrypto(cdc)
	return cdc
}
```
这个go-amino序列化的部分，只需要添加自己的模块部分即可mymodule.RegisterWire(cdc)
这个在`wire.go`里面声明定义
## 总结一遍
> （上面说的很分散，按照我说的做哈）

1. 1
2. 2
3. 3
4. 4