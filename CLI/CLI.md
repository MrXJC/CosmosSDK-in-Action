## Command-Line Interface For CosmosSDK Module
> Loc:cosmos-sdk/x/stake/client/cli/tx.go

### stake的cli源码解析
```go
func GetCmdDelegate(cdc *wire.Codec) *cobra.Command {
	cmd := &cobra.Command{
		Use:   "delegate",
		Short: "delegate coins to an existing validator/candidate",
		RunE: func(cmd *cobra.Command, args []string) error {
...
```
上面是Go语言 cobra 包 构建命令的基本结构,它主要是解析delegate命令。

* `Use` 代表子命令的名字
* `RunE` 命令的主要逻辑

```go
            amount, err := sdk.ParseCoin(viper.GetString(FlagAmount))
			if err != nil {
				return err
			}

			delegatorAddr, err := sdk.GetAddress(viper.GetString(FlagAddressDelegator))
			candidateAddr, err := sdk.GetAddress(viper.GetString(FlagAddressCandidate))
			if err != nil {
				return err
			}

			msg := stake.NewMsgDelegate(delegatorAddr, candidateAddr, amount)

			// build and sign the transaction, then broadcast to Tendermint
			ctx := context.NewCoreContextFromViper().WithDecoder(authcmd.GetAccountDecoder(cdc))

			res, err := ctx.EnsureSignBuildBroadcast(ctx.FromAddressName, msg, cdc)
			if err != nil {
				return err
			}

			fmt.Printf("Committed at block %d. Hash: %s\n", res.Height, res.Hash.String())
			return nil
		},
	}

	cmd.Flags().AddFlagSet(fsAmount)
	cmd.Flags().AddFlagSet(fsDelegator)
	cmd.Flags().AddFlagSet(fsCandidate)
	return cmd
```
* `viper.GetString`从相应的Flag字段获取字段后面的参数，以字符串读入。
* `sdk.Parsecoin` parses a cli input for one coin type
* `sdk.GetAddress` create an Address from a string
* `stake.NewMsgDelegate` 根据上面信息创建出msg
* 最后构建和签名交易，在tendermint层进行广播
* `AddFlagSet(fsAmount)` 添加命令所需参数

```go
const FlagAmount = "amount"
fsAmount   = flag.NewFlagSet("", flag.ContinueOnError)
fsAmount.String(FlagAmount, "1steak", "Amount of coins to bond")
```
`fsAmount`具体的构建流程

## gaiacli 上面添加stake的cmd

```go
// add query/post commands (custom to binary)
	rootCmd.AddCommand(
		client.GetCommands(
			...
			stakecmd.GetCmdQueryCandidate("stake", cdc),
			stakecmd.GetCmdQueryCandidates("stake", cdc),
			stakecmd.GetCmdQueryDelegatorBond("stake", cdc),
		)...)
	rootCmd.AddCommand(
		client.PostCommands(
          ...
			stakecmd.GetCmdDeclareCandidacy(cdc),
			stakecmd.GetCmdEditCandidacy(cdc),
			stakecmd.GetCmdDelegate(cdc),
			stakecmd.GetCmdUnbond(cdc),
		)...)
```

* `GetCommands()` 添加查询的命令
* `PostCommands()` 添加带参数的命令 