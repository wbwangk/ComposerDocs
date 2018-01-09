# 开发应用程序

Hyperledger Composer支持创建Web、移动或原生Node.js应用程序。它包括`composer-rest-server`（本身基于LoopBack技术）为业务网络自动生成REST API，和`hyperledger-composer`，用于生成骨架Angular应用的Yeoman框架的代码生成插件。

另外，它还包含一组丰富的JavaScript API来构建原生Node.js应用程序。

### 编写一个Node.js应用程序

[**开发Node.js应用程序与Hyperledger Composer配合**](applications_node.md)，允许你以编程方式连接到已部署的业务网络，创建、读取、更新、删除资产和参与者，以及提交交易。

### 编写Web或移动应用程序

需要与已部署的业务网络进行交互的Web或移动应用程序应调用REST API。创建REST API的最简单方法是使用`composer-rest-server`[**从已部署业务网络中的动态生成REST API**](applications_web.md)。

### 订阅事件

Node.js应用程序可以使用API`composer-client.BusinessNetworkConnection.on`调用,[**从业务网络中订阅事件**](applications_subscribing-to-events.md)。事件在业务网络模型文件中定义，并由交易处理器函数文件中的指定交易处理。

## 参考

- [**Yeoman代码生成器**](http://yeoman.io/)

- [**Angular框架**](https://angular.io/)
