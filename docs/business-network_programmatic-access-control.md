# 可编程访问控制

建议你使用声明式访问控制来实现业务网络定义中的访问控制规则。但是，你可以通过获取和测试当前参与者，在交易处理器中实现可编程的访问控制。你可以针对当前参与者的属性运行测试，以允许或拒绝交易处理器函数的执行。

## 在你开始之前

在你执行这些步骤之前，你必须对业务网络定义中的参与者进行建模，并将其部署为业务网络。你必须创建了这些参与者的一些实例，并向这些参与者发放身份。

下节的过程显示了使用下面参与者模型的示例：

```
namespace net.biz.digitalPropertyNetwork

participant Person identified by personId {
  o String personId
  o String firstName
  o String lastName
}

participant PrivilegedPerson extends Person {

}
```

## 过程

1. 在你的交易处理器函数中，使用以下`getCurrentParticipant`功能验证当前参与者的类型是否符合要求：
```
   function onPrivilegedTransaction(privilegedTransaction) {
       var currentParticipant = getCurrentParticipant();
       if (currentParticipant.getFullyQualifiedType() !== 'net.biz.digitalPropertyNetwork.PrivilegedPerson') {
           throw new Error('Transaction can only be submitted by a privileged person');
       }
       // Current participant must be a privileged person to get here.
   }
```

1. 在你的交易处理器函数中，使用以下`getCurrentParticipant`函数验证当前参与者的参与者ID ：
```
   function onPrivilegedTransaction(privilegedTransaction) {
       var currentParticipant = getCurrentParticipant();
       if (currentParticipant.getFullyQualifiedIdentifier() !== 'net.biz.digitalPropertyNetwork.Person#PERSON_1') {
           throw new Error('Transaction can only be submitted by person 1');
       }
       // Current participant must be person 1 to get here.
   }
```

可以将当前参与者的参与者ID与（通过关联）链接到资产的参与者进行比较，以验证当前参与者是否有权访问或修改某资产：
```
   function onPrivilegedTransaction(privilegedTransaction) {
       // Get the owner of the asset in the transaction.
       var assetOwner = privilegedTransaction.asset.owner;
       var currentParticipant = getCurrentParticipant();
       if (currentParticipant.getFullyQualifiedIdentifier() !== asset.owner.getFullyQualifiedIdentifier()) {
           throw new Error('Transaction can only be submitted by the owner of the asset');
       }
       // Current participant must be the owner of the asset to get here.
   }
```
