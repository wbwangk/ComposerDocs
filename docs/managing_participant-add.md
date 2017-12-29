# 添加参与者

参与者可以使用API或命令行添加到参与者库中。

## 在你开始之前

在你执行这些步骤之前，你在业务网络定义中必须建模一个参与者，并将其部署为业务网络。

下面的过程显示了一个使用以下数字财产范例业务网络定义的参与者模型的示例：[digitalproperty-network](https://www.npmjs.com/package/digitalproperty-network)

*请注意*：如果你使用`composer participant add`命令添加参与者，请确保参与者的JSON陈述包裹在单引号中。
```
namespace net.biz.digitalPropertyNetwork

participant Person identified by personId {
  o String personId
  o String firstName
  o String lastName
}
```

## 过程

1.将参与者添加到参与者库

JavaScript API:
```javascript
   const BusinessNetworkConnection = require('composer-client').BusinessNetworkConnection;
   let businessNetworkConnection = new BusinessNetworkConnection();
   return businessNetworkConnection.connect('admin@digitalPropertyNetwork')
       .then(() => {
           return businessNetworkConnection.getParticipantRegistry('net.biz.digitalPropertyNetwork');
       })
       .then((participantRegistry) => {
           let factory = businessNetworkConnection.getFactory();
           let participant = factory.newResource('net.biz.digitalPropertyNetwork', 'Person', 'mae@biznet.org');
           participant.firstName = 'Mae';
           participant.lastName = 'Smith';
           return participantRegistry.add(participant);
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
composer participant add -c admin@network -d '{"$class":"net.biz.digitalPropertyNetwork.Person","personId":"mae@biznet.org","firstName":"Mae","lastName":"Smith"}'
```
