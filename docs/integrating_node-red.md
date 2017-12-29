# 与Node-RED集成

[Node-RED](http://nodered.org/)是一个轻量级的开源集成技术，用JavaScript编写。它使用图形流程来集成不同的*节点*，节点可以接收数据、转换数据和输出数据。

Node-RED通常用于快速建立物联网样式应用的原型，或将现有的互联网服务连接在一起。

您可以使用Hyperledger Composer Node-RED来：

- 提交交易

- 读取和更新资产和参与者

- 订阅时间

- 删除资产和参与者

Hyperledger Composer Node-RED节点作为一个独立的npm包发布，发布在这里： - <https://www.npmjs.com/package/node-red-contrib-composer>

## HyperledgerComposer Node-RED节点

#### Hyperledger-Composer-out

一个节点红色输出节点，允许您创建、更新或删除资产或参与者，并提交交易。例如，将*hyperledger-composer-out*节点与*inject*节点结合使用，允许你通过提交这些参与者的JSON定义来创建参与者。

#### Hyperledger-Composer-Mid

一个节点红色mid流程节点，允许您从库中创建、获取、更新或删除资产和参与者。例如，将*hyperledger-composer-mid*与*inject*节点结合使用，运行你通过以JSON对象形式提交正确的库和身份字段来获取资产或参与者。

#### Hyperledger-Composer-In

一个Node-RED输入节点，用来订阅区块链事件。
