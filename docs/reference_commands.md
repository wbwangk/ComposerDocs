# Hyperledger Composer命令行

Hyperledger Composer命令行应用程序`composer`可用于执行多个管理、操作和开发任务。

Hyperledger Composer命令行应用程序可以使用npm进行安装：

`npm install -g composer-cli`

*请注意：使用Ubuntu时，这个命令在root用户shell中运行时会失败。*

## 业务网络档案

`composer archive create`

从磁盘上的一个目录创建业务网络档案：[composer archive create](reference/composer.archive.create.md)

`composer archive list`

验证磁盘上业务网络档案的内容：[composer archive list](reference/composer.archive.list.md)

## 业务网络卡片管理

`composer card create`

从连接配置文件、业务网络名称和证书创建业务网络卡片：[composer card create](https://hyperledger.github.io/composer/reference/composer.card.create.html)

`composer card delete`

删除本地导入的业务网络卡片：[compser card delete](https://hyperledger.github.io/composer/reference/composer.card.delete.html)

`composer card import`

将创建的卡导入你的本地钱包：[composer card import](https://hyperledger.github.io/composer/reference/composer.card.import.html)

`composer card export`

从你的钱包中导出和打包一张卡片：[composer card export](https://hyperledger.github.io/composer/reference/composer.card.export.html)

`composer card list`

列出当前在钱包中的所有卡片：[composer card list](https://hyperledger.github.io/composer/reference/composer.card.list.html)

## 业务网络管理

`composer network deploy`

部署一个业务网络定义：[composer network deploy](https://hyperledger.github.io/composer/reference/composer.network.deploy.html)

`composer network undeploy`

永久禁用一个业务网络定义：[composer network undeploy](https://hyperledger.github.io/composer/reference/composer.network.undeploy.html)

`composer network list`

列出已部署的业务网络：[composer network list](https://hyperledger.github.io/composer/reference/composer.network.list.html)

`composer network loglevel`

返回或更新composer运行时的日志级别： [composer network loglevel](https://hyperledger.github.io/composer/reference/composer.network.logLevel.html)

`composer network ping`

测试连接一个已部署的业务网络连接：[composer network ping](https://hyperledger.github.io/composer/reference/composer.network.ping.html)

`composer network update`

更新已部署的业务网络：[composer network update](https://hyperledger.github.io/composer/reference/composer.network.update.html)

`composer network upgrade`

升级一个已部署业务网络的Hyperledger Composer运行时：[composer network upgrade](https://hyperledger.github.io/composer/reference/composer.network.upgrade.html)

`composer network start`

将业务网络档案部署到一个已安装Hyperledger Composer运行时的Hyperledger Fabric peer：[composer network start](https://hyperledger.github.io/composer/reference/composer.network.start.html)

`composer runtime install`

将Hyperledger Composer运行时安装到Hyperledger Fabric peer：[composer runtime install](https://hyperledger.github.io/composer/reference/composer.runtime.install.html)

`composer network upgrade`

升级特定已部署业务网络的Hyperledger Composer运行时：[composer network upgrade](https://hyperledger.github.io/composer/reference/composer.network.upgrade.html)

## 参与者和身份管理

`composer participant add`

将参与者添加到参与者库：[composer participant add](https://hyperledger.github.io/composer/reference/composer.participant.add.html)

`composer identity issue`

向参与者发放新的身份：[composer identity issue](https://hyperledger.github.io/composer/reference/composer.identity.issue.html)

`composer identity bind`

将现有身份绑定到参与者：[composer identity bind](https://hyperledger.github.io/composer/reference/composer.identity.bind.html)

`composer identity list`

列出业务网络中的所有身份：[composer identity list](https://hyperledger.github.io/composer/reference/composer.identity.list.html)

`composer identity revoke`

撤销参与者的身份：[composer identity revoke](https://hyperledger.github.io/composer/reference/composer.identity.revoke.html)

## 交易执行

`composer transaction submit`

提交交易去执行：[composer transaction submit](https://hyperledger.github.io/composer/reference/composer.transaction.submit.html)
