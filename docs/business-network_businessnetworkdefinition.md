# 业务网络定义

业务网络定义是Hyperledger Composer编程模型的一个关键概念。它们由在`composer-common`模块中定义类`BusinessNetworkDefinition`表示，并由`composer-admin`和`composer-client`导出。

![业务网络定义图](https://hyperledger.github.io/composer/assets/img/BusinessNetworkFiles.svg)

业务网络定义由以下部分组成：

- 一组模型文件
- 一组JavaScript文件
- 一个访问控制文件

模型文件定义了业务网络的业务领域，而JavaScript文件包含交易处理器函数。交易处理器函数在Hyperledger Fabric上运行，并可访问存储在Hyperledger Fabric区块链全局状态中的资产库。

模型文件通常由业务分析师创建，因为它们定义了模型元素之间的结构和关联：资产、参与者和交易。

JavaScript文件通常由正在实现业务分析师提供的业务需求的开发人员创建。

访问控制文件包含一组访问控制规则，用于定义业务网络中不同参与者的权限。

一旦定义，可以使用`composer`命令行接口将业务网络定义打包到档案中。然后可以使用`composer-admin`模块中`AdminConnection`类在Fabric上部署、取消部署或更新这些档案。

## 参考

- [**建模语言**](reference_cto_language.md)
- [**访问控制语言**](reference_acl_language.md)
- [**交易处理器函数**](reference_js_scripts.md)
