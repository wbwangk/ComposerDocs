# Hyperledger Composer API

Hyperledger Composer是基于Hyperledger构建区块链应用的应用程序开发框架。这是Hyperledger Composer客户端、管理员和运行时JavaScript API文档。

## 快速链接

[公共API](https://hyperledger.github.io/composer/api/allData#common-api) - [客户端API](https://hyperledger.github.io/composer/api/allData#client-api) - [管理API](https://hyperledger.github.io/composer/api/allData#admin-api) - [运行时API](https://hyperledger.github.io/composer/api/allData#runtime-api)

## 概述

所有的类都在[类索引](https://hyperledger.github.io/composer/api/allData.html)中列出

Hyperledger Composer的主要组件是：

1. Hyperledger Composer语言，用于描述以区块链为后端的业务网络中的资源（资产，参与者和交易）结构。
2. JavaScript API来查询、创建、更新和删除资源，并从客户端应用程序提交交易。Hyperledger Composer资源存储在区块链中。
3. 交易提交处理时，在Hyperledger Fabric上运行的JavaScript交易处理函数。这些函数可以通过服务器端的Hyperledger Composer API更新存储在区块链上的资源的状态。

## JavaScript语言支持

使用客户端或管理员API的应用程序（不在交易函数内部运行）可以使用ES6编写。作为一个例子，它允许使用async / await语法。

```javascript
  // connect using the 'newUserCard', create an asset, add it to a registry and get all assets. 
  try{
    await businessNetworkConnection.connect('newUserCard');
    let newAsset = factory.newAsset('org.acme.sample','SampleAsset','1148');
    await assestRegistry.add(newAsset);

    result = await assetRegistry.getAll();
    LOG.info(result);

    await businessNetworkConnection.disconnect();
  } catch (error){
      // error handling
  }
```

承诺链(promise chain)语法可以使用，并且必须在交易函数中使用。

**交易函数中的所有代码必须使用ES5语法 - 以及承诺链**

对上面的例子使用承诺：
```
  // connect using the 'newUserCard', create an asset, add it to a registry and get all assets. 

  return businessNetworkConnection.connect('newUserCard')
    .then( function()  { 
        var newAsset = factory.newAsset('org.acme.sample','SampleAsset','1148');
        return assetRegistry.add(newAsset);
    })
    .then( function() {
       return assetRegistry.getAll();
    }) 
    .then( function(result) {
        LOG.info(result);
        return businessNetworkConnection.disconnect();
    })
    .catch( function (error){
      // error handling
    });
```

## 管理员和客户端API

这些API专门用于获取业务网络连接以进行管理，或获取资源以执行业务操作。业务操作可能是添加资产或提交交易函数。

这些API的起点是`AdminConnection`or `BusinessNetworkConnection`。两个API都遵循类似的连接模式。已经导入的业务网络卡片的名称是必需的。默认情况下，这些卡片可以从文件系统卡片存储准备好。

```javascript
const AdminConnection = require('composer-admin').AdminConnection;

let adminConnection = new AdminConnection();
await adminConnection.connect('cardNameToUse');

//
const BusinessNetworkConnection = require('composer-client').BusinessNetworkConnection;

let bizNetConnection = new BusinessNetworkConnection();
let businessNetworkDefintion = await bizNetConnection.connect('cardNameToUse');
```

### 库(Registry)

客户端API的关键部分是Regsitry类。有*AssetRegsitry*、*ParticipantRegistry*、*IdentityRegistry*、*TransactionRegistries*和*Historian*。这些都有一个共同的超级类型*Registry*。

Registry支持访问库以获取资源。

- 添加一个或多个资源

- 更新一个或多个资源

- 删除一个或所有资源

- 获取一个或所有资源

- 检查资源是否存在

- 解析一个或所有资源

解析与获取的区别在于，获取不会解析在资源中定义的任何关联。将提供对资源的引用。解析调用将遍历所有关联。

自动为每个资产、参与者和交易创建库(Registry)。如果需要，可以使用addRegistry调用创建其他库。

## 通用API

通用API包含的API用于获取你连接到的业务网络的信息，并创建新的资产、参与者、交易和事件。它还提供API来查找这些资源的信息。

例如创建一个新的资产
```javascript
let factory = this.businessNetworkDefinition.getFactory();
owner = factory.newResource('net.biz.digitalPropertyNetwork', 'Person', 'PID:1234567890');
owner.firstName = 'Fred';
owner.lastName = 'Bloggs';
```

### 运行时API

运行时API是所有交易函数使用的API。它允许访问API， 建立和发出查询、发出事件、获取所有类型的库、获取当前参与者、获取序列化程序从JavaScript对象创建资源、发布HTTP REST调用

与库API一起，通用API调用也可用于与资源进行交互。每个
```javascript
// Get the driver participant registry.
return getParticipantRegistry('org.acme.Driver')
  .then(function (driverParticipantRegistry) {
    // Call methods on the driver participant registry.
  })
  .catch(function (error) {
    // Add optional error handling here.
  });
```

**注意：所有的交易函数都应该用ES5编写，并附加使用承诺链**

### 交易函数

交易函数是Composer的一部分，可以认为是*智能合约*的实现。它将从客户端应用程序（或通过REST API）调用。它将操作资产状态、参与者等，底层是区块链的全局状态。交易函数所执行的操作将受制于底层区块链所确定的背书和排序协议。因此，资产*的真实来源*得以维持。

交易函数的定义在.cto模型文件中。
```
namespace net.biz.digitalPropertyNetwork

transaction RegisterPropertyForSale identified by transactionId{
  o String transactionId
  --> LandTitle title
}
```

然后通过函数注释中的注解将其链接到交易函数的实现。 `@transaction`将该函数标记为交易函数。`@param`将此交易连接到模型中定义的`RegisterPropertyForSale`。
```javascript
/**
 * @param {net.biz.digitalPropertyNetwork} registryProperty
 * @transaction
 */
function codeToImplementatTheTransactionFunction(registryProperty){
 //
}
```

要销售的函数参数`registryProperty`将是交易资源的完全解析副本。在这个例子中，`--> LandTitle title`是一个LandTitle资源的完全解析。

#### 限制

- 交易函数不应使用随机数字

- 其他交易不能从交易函数的实现中提交。可以调用其他函数，但将被视为同一交易的一部分。这与所调用函数的注解无关。

- 始终用`getCurrentParticipant()`去获取调用参与者的详细信息
