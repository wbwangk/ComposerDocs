# 编写Node.js应用程序

应用程序开发人员使用`composer-client`npm模块以编程方式连接到已部署的业务网络，创建、读取、更新、删除资产和参与者，以及提交交易。如果应用程序需要能够部署或管理业务网络，则可以使用`composer-admin`npm模块。

示例[`landregistry.js`](https://github.com/hyperledger/composer-sample-applications/blob/master/packages/digitalproperty-app/lib/landRegistry.js)文件包含一个代表土地注册的类，并包含列出土地权证、添加默认权证和提交交易的方法。这已经使用JavaScript类实现了; 然而，你可以自由地构建你的代码，如你所愿。

值得强调的是API的风格是使用承诺（promise）。通常，Hyperledger Composer API将返回在成功完成操作时解决的承诺，或返回操作结果（如果适用）。

如果你不熟悉基于Promise的开发，那么值得在线查看一些教程来获得一个想法。

## 需要的模块

```
const BusinessNetworkConnection = require('composer-client').BusinessNetworkConnection;
```

对于Hyperledger Composer客户端应用程序，这是唯一需要的npm模块。

## 连接到Hyperledger Composer运行时

BusinessNetworkConnection实例已创建，然后被用于连接到运行时：
```
this.bizNetworkConnection = new BusinessNetworkConnection();
this.cardName = config.get('cardName');
this.businessNetworkIdentifier = config.get('businessNetworkIdentifier');
```

我们要在这里创建的第一个Hyperledger Composer API调用是connect() API，用于建立与Hyperledger Fabric上的Hyperledger Composer运行时的连接。如果成功，此API将对BusinessNetworkDefinition返回一个Promise：
```
this.bizNetworkConnection.connect(this.cardName)
.then((result) => {
  this.businessNetworkDefinition = result;
});
```

对于客户端应用程序来说，这是所有必需的要点，从这个角度来看，应用程序想要做什么来调用哪些API。

## 将资产添加到库

Hyperledger Composer运行时将为每种建模资产创建一个默认库。所以在这个例子中，LandTitle（土地权证）库已经被创建了。我们在这里要做的是访问该库，然后添加一些资产。该`getAssetRegistry()`方法采用CTO模型文件中定义的全限定资产名称（即命名空间加上资产类型的名称）。它返回一个与资产库一起解决的承诺：
```
this.bizNetworkConnection.getAssetRegistry('net.biz.digitalPropertyNetwork.LandTitle')
.then((result) => {
    this.titlesRegistry = result;
});
```

下一步是创建一些资产（在代码中查找方法`_bootstrapTitles`）

工厂样式模式用于创建资产。从businessNetworkDefinition获取工厂，用于创建业务网络中定义的所有类型的实例。请注意使用命名空间和资产名称。然后我们可以设置这个资产的属性。这里的标识符（firstName lastName）与模型中定义的属性匹配。
```
let factory = this.businessNetworkDefinition.getFactory();
owner = factory.newResource('net.biz.digitalPropertyNetwork', 'Person', 'PID:1234567890');
owner.firstName = 'Fred';
owner.lastName = 'Bloggs';
```

我们现在有一个人！现在我们需要一个土地权证。请注意业主是如何被指定为我们刚刚创建的人。（在实际的示例代码中，我们使用这个代码两次来创建landTitle1和landTitle2）。
```
let landTitle2 = factory.newResource('net.biz.digitalPropertyNetwork', 'LandTitle', 'LID:6789');
landTitle2.owner = owner;
landTitle2.information = 'A small flat in the city';
```

我们现在已经创建了一个需要存储在库中的土地权证。
```
this.titlesRegistry.addAll([landTitle1, landTitle2]);
```

这是使用API来添加多个权证，这会返回一个在添加资产时解决的承诺。我们需要做的最后一件事是添加Person，Fred Bloggs。由于这是“参与者”，因此使用getParticipantRegistry API。
```
this.bizNetworkConnection.getParticipantRegistry('net.biz.digitalPropertyNetwork.Person')
  .then((personRegistry) => {
      return personRegistry.add(owner);
  })
```

## 列出库中资产

在示例应用程序中，这是以不同的方法处理的`list()`。与放置资产相同的设置是必需的，所以在我们需要获取资产库之前，我们称之为getAll() API。这将返回一个对象数组。
```
this.bizNetworkConnection.getAssetRegistry('net.biz.digitalPropertyNetwork.LandTitle')
.then((registry) => {
   return registry.getAll();
})
.then((aResources) => {
  // instantiate
  let table = new Table({
    head: ['TitleID', 'OwnerID', 'First Name', 'Surname', 'Description', 'ForSale']
  });
  let arrayLength = aResources.length;
  for(let i = 0; i < arrayLength; i++) {
    let tableLine = [];
    tableLine.push(aResources[i].titleId);
    tableLine.push(aResources[i].owner.personId);
    tableLine.push(aResources[i].owner.firstName);
    tableLine.push(aResources[i].owner.lastName);
    tableLine.push(aResources[i].information);
    tableLine.push(aResources[i].forSale ? 'Yes' : 'No');
    table.push(tableLine);
  }
  // Put to stdout - as this is really a command line app
  return(table);
})
```

其中大部分不是Hyperledger Composer API代码 - 但它显示了如何访问已返回的对象的详细信息。在这一点上，值得再看看这个模型。
```
asset LandTitle identified by titleId {
  o String   titleId
  o Person   owner
  o String   information
  o Boolean  forSale   optional
}

participant Person identified by personId {
  o String personId
  o String firstName
  o String lastName
}
```

您可以看到如何以非常简单的方式访问所有者和权证信息。

## 提交交易

我们需要做的最后一件事是提交交易。这是模型文件中交易的定义：
```
transaction RegisterPropertyForSale identified by transactionId{
  o String transactionId
  --> LandTitle title
}
```

交易在这里有两个字段，一个是trandsactionId，一个是应该提交出售的土地权证的引用。第一步是进入地产登记处，取回我们要提交出售的具体土地权证。
```
this.bizNetworkConnection.getAssetRegistry('net.biz.digitalPropertyNetwork.LandTitle')
.then((registry) => {
  return registry.get('LID:1148');
})
```

getAssetRegistry调用现在应该看起来有点熟悉，get API用于获取特定的土地权证。下一步是创建我们想要提交的交易。
```
let serializer = this.businessNetworkDefinition.getSerializer();

let resource = serializer.fromJSON({
  '$class': 'net.biz.digitalPropertyNetwork.RegisterPropertyForSale',
  'title': 'LID:1148'
});

return this.bizNetworkConnection.submitTransaction(resource);
```

我们需要做的是创建一个“序列化器”。这是能够创建一个资源 - 这个资源随后被传递给submitTransaction API。请注意，交易JSON与模型文件中指定的结构要匹配。

## 参考

- [**JavaScript API文档**](https://hyperledger.github.io/composer/api/api-doc-index.html)
- [**承诺教程**](https://scotch.io/tutorials/understanding-javascript-promises-pt-i-background-basics)
