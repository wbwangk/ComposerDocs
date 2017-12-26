# 订阅事件

Node.js应用程序可以使用`composer-client.BusinessNetworkConnection.on`API调用从业务网络订阅事件。事件在业务网络模型文件中定义，并由交易处理函数文件中的指定交易处理。有关发布事件的更多信息，请参阅[发布事件](business-network_publishing-events.md)。

## 在你开始之前

在应用程序可以订阅事件之前，你必须定义一些事件和发送它们的交易。还必须部署业务网络，并且必须具有可连接到该业务网络的连接配置文件。

## 过程

1. 应用程序必须发送特定的API调用来订阅业务网络中事件发出的交易。目前，订阅事件的应用程序将接收所有发送的事件。API调用应采用以下格式：
```
businessNetworkConnection.on('event', (event) => {
    // event: { "$class": "org.namespace.BasicEvent", "eventId": "0000-0000-0000-000000#0" }
    console.log(event);
});
```

这包括在[发布事件](business-network_publishing-events.md)文档中创建的一个叫`BasicEvent`的事件。该`eventId`属性始终与发送事件的交易的`transactionId`相同，并`以"transactionId": "<transactionId>#number"`的格式附加了一个数字。

## 接下来是什么？

应用程序现在会收到业务网络发出的所有事件，并由应用程序决定这些集成事件的范围。
