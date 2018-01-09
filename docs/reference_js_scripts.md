# 交易处理器函数

Hyperledger Composer业务网络定义由一组模型文件和一组脚本组成。这些脚本可能包含交易处理器函数，这些函数可以实现业务网络定义的模型文件中定义的交易。

当使用BusinessNetworkConnection API提交交易时，交易处理函数将由运行时自动调用。

带有文档注释的装饰器在运行时处理时，使用元数据对函数进行注解。

每种交易类型都有一个关联的存储交易的库。

## 交易处理器结构

交易处理器函数的结构包括装饰器和元数据，后面跟着一个JavaScript函数，这两个部分都是交易处理器函数工作所必需的。

交易处理器函数之上的第一行注释包含交易处理器函数所做的人类可读描述。第二行必须包含`@param`标签来指示参数定义。该`@param`标签之后是触发交易处理器函数交易的资源名称，这需要业务网络的命名空间的格式，其次是交易的名称。资源名称后面是引用资源的参数名称，该参数必须作为参数提供给JavaScript函数。第三行必须包含`@transaction`标签，该标签将代码标识为交易处理器函数因而是必需的。
```
/**
* A transaction processor function description
* @param {org.example.sampleTransaction} parameter-name A human description of the parameter
* @transaction
*/
```

在注释之后是为交易提供价值的JavaScript函数。该函数可以具有任何名称，但必须包含在注解中定义的参数名称作为参数。
```
function transactionProcessor(parameter-name) {
  //Do some things.
}
```

完整的交易处理器函数将采用以下格式：
```
/**
* A transaction processor function description
* @param {org.example.sampleTransaction} parameter-name A human description of the parameter
* @transaction
*/
function transactionProcessor(parameter-name) {
  //Do some things.
}
```

## 编写交易处理器函数

交易处理函数是在模型文件中定义的交易的逻辑操作。例如，交易的交易处理器函数`Trade`可以使用JavaScript将`owner`资产的属性从一个参与者改变到另一个参与者。

下面是一个例子`basic-sample-network`，下面的`SampleAsset`定义包含一个名为`value`的属性，它被定义为一个字符串。该`SampleTransaction`交易需要一个到资产的关联，资产会被改变，`value`属性的新值由交易的属性`newValue`提供。
```
asset SampleAsset identified by assetId {
  o String assetId
  --> SampleParticipant owner
  o String value
}

transaction SampleTransaction {
  --> SampleAsset asset
  o String newValue
}
```

与交易`SampleTransaction`关联的交易处理器函数是使资产和资产存储库都发生变化的原因。

交易处理函数将`SampleTransaction`类型定义为关联交易，并将其定义为参数`tx`。然后，它保存要由交易更改的资产的原始值，将其替换为在提交交易期间传递的值（`newValue`交易定义中的属性），更新库中的资产，然后发出事件。
```
/**
 * Sample transaction processor function.
 * @param {org.acme.sample.SampleTransaction} tx The sample transaction instance.
 * @transaction
 */
function sampleTransaction(tx) {

    // Save the old value of the asset.
    var oldValue = tx.asset.value;

    // Update the asset with the new value.
    tx.asset.value = tx.newValue;

    // Get the asset registry for the asset.
    return getAssetRegistry('org.acme.sample.SampleAsset')
        .then(function (assetRegistry) {

            // Update the asset in the asset registry.
            return assetRegistry.update(tx.asset);

        })
        .then(function () {

            // Emit an event for the modified asset.
            var event = getFactory().newEvent('org.acme.sample', 'SampleEvent');
            event.asset = tx.asset;
            event.oldValue = oldValue;
            event.newValue = tx.newValue;
            emit(event);

        });

}
```

## 交易处理函数中的错误处理

交易处理器函数将失败，并回滚任何已经发生错误的更改。整个交易失败，不仅仅是交易处理，交易处理器函数在错误发生之前改变的任何东西都将被回滚。
```
/**
 * Sample transaction processor function.
 * @param {org.acme.sample.SampleTransaction} tx The sample transaction instance.
 * @transaction
 */
function sampleTransaction(tx) {
    // Do something.
    throw new Error('example error');
    // Execution stops at this point; the transaction fails and rolls back.
    // Any updates made by the transaction processor function are discarded.
    // Transaction processor functions are atomic; all changes are committed,
    // or no changes are committed.
}
```

交易所做的更改是原子操作，要么交易成功且生效所有更改，或者交易失败且所有更改无效。

## 解析交易中的关联

当一个交易涉及的资产、交易或参与者的某属性是一个关联时，关联会自动解析。所有关联，包括嵌套关联，会在交易处理函数运行之前解析。

以下示例包含嵌套关联，交易含有一个到资产的关联，这个资产又含有一个到参与者的关联，因为所有关联都已解析，资产的`owner`属性被解析为指定的参与者。

模型文件：
```
namespace org.acme.sample

participant SampleParticipant identified by participantId {
  o String participantId
}

asset SampleAsset identified by assetId {
  o String assetId
  --> SampleParticipant owner
}

transaction SampleTransaction {
  --> SampleAsset asset
}
```

脚本文件：
```javascript
/**
 * Sample transaction processor function.
 * @param {org.acme.sample.SampleTransaction} tx The sample transaction instance.
 * @transaction
 */
function sampleTransaction(tx) {
    // The relationships in the transaction are automatically resolved.
    // This means that the asset can be accessed in the transaction instance.
    var asset = tx.asset;
    // The relationships are fully or recursively resolved, so you can also
    // access nested relationships. This means that you can also access the
    // owner of the asset.
    var owner = tx.asset.owner;
}
```

在这个例子中，不仅可以在交易中通过关联使用`tx.asset`引用特定的资产，还可以通过关联使用`tx.asset.owner`引用特定的参与者`owner`。在这种情况下，`tx.asset.owner`将被解析为引用一个特定的参与者。

## 交易处理器函数中的Promise返回

与关联类似，交易处理器函数将在提交交易之前等待承诺解决。如果承诺被拒绝，交易将失败。

在下面的示例代码中有几个承诺，在每个承诺都返回前交易将不会完成。

模型文件：
```
namespace org.acme.sample

transaction SampleTransaction {

}
```

脚本文件：
```javascript
/**
 * Sample transaction processor function.
 * @param {org.acme.sample.SampleTransaction} tx The sample transaction instance.
 * @transaction
 */
function sampleTransaction(tx) {
    // Transaction processor functions can return promises; Composer will wait
    // for the promise to be resolved before committing the transaction.
    // Do something that returns a promise.
    return Promise.resolve()
        .then(function () {
            // Do something else that returns a promise.
            return Promise.resolve();
        })
        .then(function () {
            // Do something else that returns a promise.
            // This transaction is complete only when this
            // promise is resolved.
            return Promise.resolve();
        });
}
```

## 在交易处理器函数中使用API

Hyperledger Composer API可以在交易处理函数中调用，在下面的代码示例中，`getAssetRegistry`调用会返回一个在交易完成之前解决的承诺。

模型文件：
```
namespace org.acme.sample

asset SampleAsset identified by assetId {
  o String assetId
  o String value
}

transaction SampleTransaction {
  --> SampleAsset asset
  o String newValue
}
```

脚本文件：
```javascript
/**
 * Sample transaction processor function.
 * @param {org.acme.sample.SampleTransaction} tx The sample transaction instance.
 * @transaction
 */
function sampleTransaction(tx) {
    // Update the value in the asset.
    var asset = tx.asset;
    asset.value = tx.newValue;
    // Get the asset registry that stores the assets. Note that
    // getAssetRegistry() returns a promise, so we have to return
    // the promise so that Composer waits for it to be resolved.
    return getAssetRegistry('org.acme.sample.SampleAsset')
        .then(function (assetRegistry) {
            // Update the asset in the asset registry. Again, note
            // that update() returns a promise, so so we have to return
            // the promise so that Composer waits for it to be resolved.
            return assetRegistry.update(asset);
        })
}
```

## 接下来是什么？

交易处理函数也可以用来：

- [**定义查询**](business-network_query.md)以从couchDB数据库获取有关区块链世界状态的信息。

- [**定义**](business-network_publishing-events.md)将事件数据发送给应用程序的事件。

