# Architecture


1. **types.go** 定义了一些此模块需要的结构，对于stake需要定义validator，delegator（除此之外他还特地新建了pool.go,来定义抵押池，实际上也可以放入types）
2. **msg.go** 定义了此模块的接受的客户端交易信息的数据格式。对于stake来说就只有四种交易信息 
  * 申请成为验证信息
  * 修改验证节点信息
  * 向验证节点抵押贷币
  * 从验证节点解绑贷币
3. **keeper.go** 主要是对程序状态里面的信息在KVstore中存取
4. **handle.go** 根据msg的类型实现代码逻辑，一般就是实现abci接口的 checktx 和 delivertx。对于stake来说，也就是四种方法。
