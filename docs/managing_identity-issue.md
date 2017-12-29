# 向参与者发放新的身份

使用API，命令行或在Hyperledger Composer Playground中使用ID卡，可以向参与者发放新的身份。一旦发放了新的身份，参与者就可以使用该身份在参与者上下文中与业务网络进行交互。

使用Hyperledger Fabric时，Hyperledger Composer使用Hyperledger Fabric证书颁发机构（CA）来注册新的登记证书，从而发放新的身份。Hyperledger Fabric证书颁发机构生成一个可以提供给参与者的登记密码，然后可以使用登记密码从Hyperledger Fabric证书颁发机构请求登记证书和私钥。

## 在你开始之前

在执行这些步骤之前，你必须已将参与者添加到参与者库中。新身份的**发放人**（无论使用命令行或使用下面的JavaScript API）本身必须具有“发放人”的权威和适当的ACL，ALC要允许他们Hyperledger Composer中发放（与参与者相关联的）身份。

下面的过程显示了一个使用以下数字财产范例业务网络定义的参与者模型的示例：[digitalproperty-network](https://www.npmjs.com/package/digitalproperty-network)
```
namespace net.biz.digitalPropertyNetwork

participant Person identified by personId {
  o String personId
  o String firstName
  o String lastName
}
```

该示例假定该`net.biz.digitalPropertyNetwork#mae@biznet.org`参与者的实例已经被创建并被放入参与者库中。

## 过程

1.连接到业务网络并向参与者颁发新的身份

JavaScript API:
```javascript
  const BusinessNetworkConnection = require('composer-client').BusinessNetworkConnection;
  let businessNetworkConnection = new BusinessNetworkConnection();
  return businessNetworkConnection.connect('admin@digitalPropertyNetwork')
      .then(() => {
          return businessNetworkConnection.issueIdentity('net.biz.digitalPropertyNetwork.Person#mae@biznet.org', 'maeid1')
      })
      .then((result) => {
          console.log(`userID = ${result.userID}`);
          console.log(`userSecret = ${result.userSecret}`);
          return businessNetworkConnection.disconnect();
      })
      .catch((error) => {
          console.error(error);
          process.exit(1);
      });

```

命令行:
```bash
  composer identity issue -c admin@network -f maeid1.card -u maeid1 -a "resource:net.biz.digitalPropertyNetwork.Person#mae@biznet.org"
```

1.作为参与者，测试与业务网络的连接

JavaScript API:
```javascript
  const BusinessNetworkConnection = require('composer-client').BusinessNetworkConnection;
  let businessNetworkConnection = new BusinessNetworkConnection();
  return businessNetworkConnection.connect('admin@digitalPropertyNetwork')
      .then(() => {
          return businessNetworkConnection.ping();
      })
      .then((result) => {
          console.log(`participant = ${result.participant ? result.participant : '<no participant found>'}`);
          return businessNetworkConnection.disconnect();
      })
      .catch((error) => {
          console.error(error);
          process.exit(1);
      });
```
命令行:
```bash
  composer network ping -c maeid1@network
```
