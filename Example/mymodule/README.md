#MyModule
>简单的自己应用业务逻辑的实现。

##Description
实现了很简单的逻辑，这个模块接收两种消息`MsgDo`和`MsgUndo`,前者消息的操作是根据提供的非负整数N，对发送消息的账户，增加10steak*N的token，后者则是撤销由`MsgDo`消息导致账户累计增加的全部token。具体细节可以见Usage.
##Install

```bash
cd $GOPATH/src/github.com
mkdir cosmos
cd cosmos
git clone https://github.com/cosmos/cosmos-sdk.git
```
克隆cosmos-sdk源码

```bash
cd $GOPATH/src/github.com
mkdir mrxjc
cd mrxjc
git clone https://github.com/MrXJC/CosmosSDK-in-Action.git
```
克隆本仓库源码

```bash
cd $GOPATH/src/github.com/cosmos/cosmos-sdk
git checkout v0.17.0
cp -rf $GOPATH/src/github.com/mrxjc/CosmosSDK-in-Action/Example/mymodule/x/mymodule x/mymodule
cp -rf $GOPATH/src/github.com/mrxjc/CosmosSDK-in-Action/Example/mymodule/cmd/gaia/app/app.go  cmd/gaia/app/
cp -rf $GOPATH/src/github.com/mrxjc/CosmosSDK-in-Action/Example/mymodule/cmd/gaia/cmd/gaiacli/main.go  cmd/gaia/cmd/gaiacli/
```
把Example中的mymodule源码都拷贝到cosmos-sdk里面。

```
dep ensure
make install
```
 编译安装，生成gaiad与gaiacli。
 
##Usage
```bash
gaiad init gen-tx --name=gov1 --home="gaia1"
{
  "app_message": {
    "secret": "siege brief foam drive side oak strong swear evoke clutch business uphold giraffe lava assume abandon"
  },
  "gen_tx_file": {
    "node_id": "7d8e637b63a16099b36a75b0afe5a31a5a3515df",
    "ip": "192.168.150.109",
    "validator": {
      "pub_key": {
        "type": "AC26791624DE60",
        "value": "/25u3/ds1UpMDmpGrMs/SHr+9JwVPi4ms26egL602FY="
      },
      "power": 100,
      "name": ""
    },
    "app_gen_tx": {
      "name": "gov1",
      "address": "D2A7EAFFAC63040166C1FB669D2B32CEC40240EB",
      "pub_key": {
        "type": "AC26791624DE60",
        "value": "/25u3/ds1UpMDmpGrMs/SHr+9JwVPi4ms26egL602FY="
      }
    }
  }
}
```
```bash
gaiad init --gen-txs --chain-id=gov-test -o --home=gaia1                                              
{
  "chain_id": "gov-test",
  "node_id": "7d8e637b63a16099b36a75b0afe5a31a5a3515df",
  "app_message": null
}
```
构建测试网

```bash
gaiacli keys list                                                                           
NAME:	ADDRESS:					PUBKEY:
gov1	D2A7EAFFAC63040166C1FB669D2B32CEC40240EB	1624DE62203EB89AB4E005EC3DCF348466895583713CB2EFEC452A42D4C0F9697DAE3B824C
```
查询当前测试网的账户，其实这个账户就是创建节点的初始用户，他的地址就是`D2A7EAFFAC63040166C1FB669D2B32CEC40240EB`,默认有50个steak.

```bash
gaiad start --home=gaia1
```
启动节点gov1



```bash
VADDR1=D2A7EAFFAC63040166C1FB669D2B32CEC40240EB
```
```
```
查询账户


```bash
gaiacli do $VADDR1 5 --name=gov1 --chain-id=gov-test
gaiacli account $VADDR1                                                             
{
  "type": "6C54F73C9F2E08",
  "value": {
    "address": "D2A7EAFFAC63040166C1FB669D2B32CEC40240EB",
    "coins": [
      {
        "denom": "gov1Token",
        "amount": 1000
      },
      {
        "denom": "steak",
        "amount": 100
      }
    ],
    "public_key": {
      "type": "AC26791624DE60",
      "value": "PriatOAF7D3PNIRmiVWDcTyy7+xFKkLUwPlpfa47gkw="
    },
    "sequence": 1
  }
}
```
发送`MsgDo`消息
查询账户余额为70steak,因为50steak+5*10steak，一共是100steak

```bash
gaiacli account $VADDR1                                                              
{
  "type": "6C54F73C9F2E08",
  "value": {
    "address": "D2A7EAFFAC63040166C1FB669D2B32CEC40240EB",
    "coins": [
      {
        "denom": "gov1Token",
        "amount": 1000
      },
      {
        "denom": "steak",
        "amount": 150
      }
    ],
    "public_key": {
      "type": "AC26791624DE60",
      "value": "PriatOAF7D3PNIRmiVWDcTyy7+xFKkLUwPlpfa47gkw="
    },
    "sequence": 2
  }
}
```
再次发送`MsgDo`消息
查询账户余额为150steak,因为100steak+5*10steak，一共是150steak

```bash
gaiacli undo $VADDR1 --name=gov1 --chain-id=gov-test
gaiacli account $VADDR1                                                              
{
  "type": "6C54F73C9F2E08",
  "value": {
    "address": "D2A7EAFFAC63040166C1FB669D2B32CEC40240EB",
    "coins": [
      {
        "denom": "gov1Token",
        "amount": 1000
      },
      {
        "denom": "steak",
        "amount": 50
      }
    ],
    "public_key": {
      "type": "AC26791624DE60",
      "value": "PriatOAF7D3PNIRmiVWDcTyy7+xFKkLUwPlpfa47gkw="
    },
    "sequence": 2
  }
}

```
发送`MsgUndo`消息
查询账户余额为150steak,因为150steak-50steak-50steak，最后剩50steak

