# Hyperledger Composer历史库

Hyperledger Composer Historian是一个专门记录成功交易的库，包括提交他们的参与者和身份。历史库将交易存储为在Hyperledger Composer系统命名空间中定义的一个`HistorianRecord`资产。

历史库是Hyperledger Composer系统级实体。为了将历史库作为访问控制的资源，历史库必须被引用为：`org.hyperledger.composer.system.HistorianRecord`。

**请注意：**所有参与者必须有创建`HistorianRecord`资产的权限。如果交易是由无权创建`HistorianRecord`资产的参与者提交的，则交易将失败。

## HistorianRecord资产

历史库将成功的交易存储为`HistorianRecord`资产。只要交易成功完成，`HistorianRecord`就会创建一个资产并添加到历史库中。Record资产在系统命名空间中定义，并具有以下定义：
```
asset HistorianRecord identified by transactionId {
  o String      transactionId
  o String      transactionType
  --> Transaction transactionInvoked
  --> Participant participantInvoking  optional
  --> Identity    identityUsed         optional
  o Event[]       eventsEmitted        optional
  o DateTime      transactionTimestamp
}
```

- `String transactionId`，创建`HistorianRecord`资产的交易的transactionId 。
- `String transactionType`，创建`HistorianRecord`资产的交易类别。
- `Transaction transactionInvoked`，一个创建`HistorianRecord`资产的交易的关联。
- `Participant participantInvoking`， 一个指向提交交易的参与者的关联。
- `Identity identityUsed` ，一个指向提交交易的身份的关联。
- `Event[] eventsEmitted` ，一个包含交易发出的任何事件的可选属性。
- `DateTime transactionTimestamp`，创建`HistorianRecord`资产的交易的时间戳。

所有`HistorianRecord`资产都有关联指向创建它们的交易、调用交易的参与者和提交交易时使用的身份。希望获得这些属性的应用程序必须解析这种关联。

## 系统交易

Hyperledger Composer运行时所做的几个操作被归类为交易。这些“系统交易”是在Hyperledger Composer系统模型中定义的。以下将添加`HistorianRecord`资产：

- 添加、删除和更新资产

- 添加、删除和更新参与者

- 签发、绑定、激活和撤销身份

- 更新业务网络定义

## 保护历史库数据

作为一个库(registry)，访问历史库数据可以通过访问控制规则来控制。但是，作为系统级实体，历史库的资源名称始终是`org.hyperledger.composer.system.HistorianRecord`。

以下访问控制规则让成员只能看到自己提交交易的历史库数据。
```
rule historianAccess{
  description: "Only allow members to read historian records referencing transactions they submitted."
  participant(p): "org.example.member"
  operation: READ
  resource(r): "org.hyperledger.composer.system.HistorianRecord"
  condition: (r.participantInvoking.getIdentifier() == p.getIdentifier())
  action: ALLOW

}
```

## 获取历史库数据

历史库的数据可以使用API调用或查询来获取。

### 在历史库中使用客户端和REST API

`HistorianRecord`资产可以使用REST API 调用`system/historian`和`system/historian/{id}`返回。

在使用REST API时，GET调用`system/historian`将返回*所有*历史数据。这个调用应该小心使用，返回不受限制，可能会导致大量的数据被返回。

`system/historian/{id}`使用REST API的GET调用将返回`HistorianRecord`指定的资产。

### 查询历史库

历史库可以像其他库(registry)一样查询。例如，返回所有`HistorianRecord`资产的典型查询如下所示：
```
    .then(() => {       
        return businessNetworkConnection.getHistorian();
    }).then((historian) => {
        return historian.getAll();
    }).then((historianRecords) => {        
        console.log(prettyoutput(historianRecords));
    })
```

由于这是一个“getAll”调用，它可能会返回大量的数据。因此，查询部分记录的能力很重要。一个典型的例子就是根据时间来选择记录。这使用了按交易时间戳（超过某时间点）选择记录的查询能力。返回的记录可以用相同的方式处理。
```javascript
  let now = new Date();
  now.setMinutes(10);  // set the date to be time you want to query from

  let q1 = businessNetworkConnection.buildQuery('SELECT org.hyperledger.composer.system.HistorianRecord ' +
                                                'WHERE (transactionTimestamp > _$justnow)');   

  return businessNetworkConnection.query(q1,{justnow:now});
```

可以使用更高级的查询; 例如，以下查询选择并返回“添加”、“更新”和“移除”资产系统交易。
```
  // build the special query for historian records
  let q1 = businessNetworkConnection.buildQuery(
      `SELECT org.hyperledger.composer.system.HistorianRecord
          WHERE (transactionType == 'AddAsset' OR transactionType == 'UpdateAsset' OR transactionType == 'RemoveAsset')`
  );      

  return businessNetworkConnection.query(q1);
```

## 接下来是什么？

- [将查询应用到业务网络。](business-network_query.md)

- [从交易中发送事件。](business-network_publishing-events.md)

- [Hyperledger Composer API文档。](api/api-doc-index.md)

