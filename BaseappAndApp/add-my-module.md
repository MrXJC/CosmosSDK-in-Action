# How to Add My Module
> 介绍基于gaia应用基础上添加一个模块的顺序。

1. 在x(eXten)sion下面新建一个template文件夹。
2. 再在template文件夹下实现types.go,msg.go,keeper.go,handler.go,wire.go
....
3. 再把上面`.go`文件中定义的结构和接口在app.go中注册。

具体细节我会在再下几章介绍。