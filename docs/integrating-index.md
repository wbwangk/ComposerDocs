# 与现有系统集成

Hyperledger Composer可以通过使用Loopback API与现有系统集成。集成现有系统使你可以从现有业务系统中提取数据，并将其转换为Composer业务网络中的资产或参与者。

### 生成一个REST API

Hyperledger Composer包含独立的[将业务网络暴露为REST API的Node.js进程](https://hyperledger.github.io/composer/stable/integrating/getting-started-rest-api.html)。LoopBack框架用于生成由Swagger文档描述的Open API。

### 从REST服务器发布事件

REST服务器可以[**配置为订阅**](https://hyperledger.github.io/composer/stable/integrating/publishing-events.html)从已部署的业务网络发出的事件，并将这些事件发布到客户端应用程序。

### 为REST服务器启用身份验证

REST服务器可以[**配置为对客户端进行身份验证**](https://hyperledger.github.io/composer/stable/integrating/enabling-rest-authentication.html)。启用此选项时，客户端必须在允许调用REST API之前向REST服务器进行身份认证。

### 为REST服务器启用多用户模式

REST服务器可以[**配置为多用户模式**](https://hyperledger.github.io/composer/stable/integrating/enabling-multiuser.html)。多用户模式要求REST服务器的客户端为了数字签名事务，提供他们自己的区块链身份。这使业务网络能够区分REST服务器的不同客户端。

### 使用HTTPS和TLS来保护REST服务器

在生产环境中部署Hyperledger Composer REST服务器时，应将REST服务器[**配置为使用HTTPS和TLS**](https://hyperledger.github.io/composer/stable/integrating/securing-the-rest-server.html)（传输层安全性）进行保护。一旦REST服务器配置了HTTPS和TLS，在REST服务器和所有REST客户端之间传输的所有数据都将被加密。

### 为业务网络部署REST服务器

通过为业务网络部署REST服务器，你可以[**将现有系统和数据与Hyperledger Composer业务网络集成**](https://hyperledger.github.io/composer/stable/integrating/deploying-the-rest-server.html)，允许你创建、更新或删除资产和参与者，以及获取和提交交易。

### 与Node-RED集成

[Node-RED](http://nodered.org/)包括许多[**Hyperledger Composer 节点，允许你提交事务，读取，更新和删除资产和参与者，以及订阅事件。**](https://hyperledger.github.io/composer/stable/integrating/node-red.html)

### 调用外部REST服务

[**事务处理器函数可以用来调用外部REST服务**](https://hyperledger.github.io/composer/stable/integrating/call-out.html)。这使你可以将区块链中的复杂计算移出。

有关设置Loopback API的说明，请参阅[生成REST API](https://hyperledger.github.io/composer/stable/integrating/getting-started-rest-api.html)。
