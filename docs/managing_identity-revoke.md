# 撤销参与者的身份

可以使用API或命令行从参与者中撤销身份。一旦身份被撤销，参与者就不能再使用该身份与该参与者上下文中的商业网络进行交互。

使用Hyperledger Fabric时，Hyperledger Composer当前不会尝试使用Hyperledger Fabric证书发放机构（CA）API撤销身份。身份仍然可以用于向底层区块链网络提交交易，但交易将被部署的业务网络拒绝。

## 在你开始之前

在执行这些步骤之前，您必须已经将参与者添加到参与者库中，并向该参与者发放或绑定了身份。您还必须在身份库中找到该身份的唯一标识符。有关查找身份的唯一标识符的更多信息，请查看[列出业务网络中的所有身份](managing_identity-list.md)。

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

该示例还假设已经向该参与者发放了一个`maeid1`身份，并且该身份的唯一标识符是“f1c5b9fe136d7f2d31b927e0dcb745499aa039b201f83fe34e243f36e1984862”。

## 过程

1.连接到业务网络并撤消参与者的现有身份

JavaScript API:
```javascript
   const BusinessNetworkConnection = require('composer-client').BusinessNetworkConnection;
   let businessNetworkConnection = new BusinessNetworkConnection();
   return businessNetworkConnection.connect('admin@digitalPropertyNetwork')
       .then(() => {
           return businessNetworkConnection.revokeIdentity('f1c5b9fe136d7f2d31b927e0dcb745499aa039b201f83fe34e243f36e1984862')
       })
       .then(() => {
           return businessNetworkConnection.disconnect();
       })
       .catch((error) => {
           console.error(error);
           process.exit(1);
       });
```

命令行:
```bash
composer identity revoke -c admin@digitalPropertyNetwork -u f1c5b9fe136d7f2d31b927e0dcb745499aa039b201f83fe34e243f36e1984862
```
