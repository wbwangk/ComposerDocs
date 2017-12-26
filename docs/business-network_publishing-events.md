# 发送事件

事件可以由Hyperledger Composer发出并由外部应用程序订阅。事件在业务网络定义的模型文件中定义，并由交易处理器函数文件中的交易JavaScript发出。

## 在你开始之前

在开始将事件添加到您的业务网络之前，您应该对业务网络的建模语言以及构成完整的业务网络定义的内容有深入的了解。

## 过程

1. 事件在业务网络定义的模型文件（`.cto`）中定义，与资产和参与者相同。事件使用以下格式：
   ```
   event BasicEvent {
   }
   ```

2. 为了事件被发布，创建事件的交易必须调用三个函数，第一个是`getFactory`函数。`getFactory`允许事件作为交易的一部分而被创建。接下来，必须使用`factory.newEvent('org.namespace', 'BasicEvent')`创建一个事件。这将在指定的命名空间中创建一个`BasicEvent`事件。然后必须设置事件所需的属性。最后，事件必须通过使用`emit(BasicEvent)`发送。调用这个事件的简单交易将如下所示：
   ```javascript
   /**
    * @param {org.namespace.BasicEventTransaction} basicEventTransaction
    * @transaction
    */
   function basicEventTransaction(basicEventTransaction) {
       var factory = getFactory();

       var basicEvent = factory.newEvent('org.namespace', 'BasicEvent');
       emit(basicEvent);
   }
   ```

此交易创建并发出在业务网络模型文件中定义的`BasicEvent`类型的事件。有关getFactory函数的更多信息，请参阅[Composer API文档](https://hyperledger.github.io/composer/jsdoc/module-composer-runtime.html#getFactory)。

## 接下来是什么？

- [订阅活动](https://hyperledger.github.io/composer/applications/subscribing-to-events.html)
- [开发应用程序](https://hyperledger.github.io/composer/applications/applications-index.html)
