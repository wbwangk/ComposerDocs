# 管理一个已部署的业务网络

### 连接器的具体信息

### 参与者和身份

参与者和身份是Hyperledger Composer的核心概念。参与者是业务网络的成员，可能代表个人或组织。参与者有身份文档可以证明自己的身份。欲了解更多信息，请参阅[参与者和身份](managing_participantsandidentities.md)。

### 添加参与者

[参与者必须添加到业务网络](managing_participant-add.md)才能进行交易。参与者可以创建资产，并与其他参与者交换资产。参与者通过提交交易来处理资产。

### 创建、导出和导入业务网络卡片

业务网络卡片由连接profile、身份和证书组成，在Hyperledger Composer Playground中通过它才允许连接到业务网络。可以从Hyperledger Composer Playground 的我的钱包页面[创建、导出和导入](managing_id-cards-playground.md)业务网络卡片。

### 向参与者发放新的身份

[使用API或命令行向参与者发放新的身份](managing_identity-issue.md)。一旦发放了新的身份，参与者就可以使用该身份在其上下文中与业务网络进行交互。

### 将现有身份绑定到参与者

[使用API或命令行可以将现有的身份绑定到参与者](managing_identity-issue.md)。一旦已经存在的身份被绑定，参与者就可以使用该身份在其上下文中与业务网络进行交互。

### 列出业务网络中的所有身份
为了在已部署的业务网络中执行身份管理操作，你可以[通过API或命令行列出和查看身份库中的一组身份](managing_identity-list.md)。

### 撤销参与者的身份

[可以使用API或命令行从参与者中撤销身份](managing_identity-revoke.md)。一旦身份被撤销，参与者就不能再使用该身份在其上下文中与业务网络进行交互。

### 更新Hyperledger Composer运行时

要将[Hyperledger Composer更新](managing_updating-composer.md)为新版本，必须使用npm卸载并重新安装Hyperledger Composer组件。

## 接下来是什么？

- 你可能希望使用LoopBack将Hyperledger Composer  [与现有系统集成](integrating_integrating-index.md)。

- 从你的业务网络的消费数据的应用程序可以[订阅事件](applications_subscribing-to-events.md)。

