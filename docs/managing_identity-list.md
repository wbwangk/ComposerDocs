# 列出业务网络中的所有身份

当向参与者发放新身份或者将现有身份绑定到参与者时，在已部署的业务网络中的身份库中一个身份与参与者之间的映射被创建。当该参与者使用该身份将事务提交到已部署的业务网络时，Composer运行时会在身份库中查找该身份的有效映射。这种查找是使用公钥签名或指纹完成的，指纹本质上是证书内容的散列（对证书和身份唯一的）。

为了在已部署的业务网络中执行身份管理操作，你需要列出和查看身份库中的一组身份。

## 在你开始之前

在执行这些步骤之前，你应该已经将参与者添加到参与者库，并向该参与者发放了新的身份或绑定了现有的身份。否则身份库将是空的，你不会看到任何结果。

## 过程

1.连接到业务网络并在身份库中列出身份

JavaScript API:
```javascript
  const BusinessNetworkConnection = require('composer-client').BusinessNetworkConnection;
  let businessNetworkConnection = new BusinessNetworkConnection();
  return businessNetworkConnection.connect('admin@digitalPropertyNetwork')
      .then(() => {
          return businessNetworkConnection.getIdentityRegistry();
      })
      .then((identityRegistry) => {
          return identityRegistry.getAll();
      })
      .then((identities) => {
          identities.forEach((identity) => {
            console.log(`identityId = ${identity.identityId}, name = ${identity.name}, state = ${identity.state}`);
          });
          return businessNetworkConnection.disconnect();
      })
      .catch((error) => {
          console.error(error);
          process.exit(1);
      });
```

命令行:
```bash
composer identity list -c admin@digitalPropertyNetwork
```
