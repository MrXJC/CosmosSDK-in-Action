# How to add my module
> 介绍基于gaia应用基础上添加一个模块的顺序。

1. 在x(eXten)sion下面新建一个template文件夹。
2. 再在template文件夹下实现types.go,msg.go,keeper.go,handler.go,wire.go
....
3. 再他们在app.go中注册。