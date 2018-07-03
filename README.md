# Introduction

> How to write the module using CosmosSDK v0.17.0 based on Gaia such as bank, stake, governance.
> [**doc**](https://mikexu.gitbook.io/cosmossdk-in-action/)

## Outline
主要介绍Tendermint和CosmosSDK是什么。ABCI是什么。ABCI在Tendermint和CosmosSDK中间起到了什么作用。整个交易信息如何在tendermint和CosmosSDK中间流通

## Module Create
分模块介绍了构建CosmosSDK模块方法与顺序，如何编写模块的代码。主要以stake为例子，一边进行源码分析，一边进行源码模版的总结,从零开发MyModule，提供开发者参考。

## Module CLI
介绍模块的命令行接口，发送交易和查询交易的细节。

##  Example
* [**MyModule**](https://github.com/MrXJC/CosmosSDK-in-Action/tree/master/Example/mymodule)从零(gaia)开始构建一个简单的ABCI应用来开发区块链应用

## Module Test

## Appendix
ComosSDK和Tendermint中比较常用的通用的结构和接口的分析。
