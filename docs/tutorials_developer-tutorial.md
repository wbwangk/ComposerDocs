## 创建一个Hyperledger Composer解决方案的开发者教程

本教程将引导你从头开始构建Hyperledger Composer区块链解决方案。在几个小时的时间里，你将能够从破坏性区块链创新的想法转变为针对真正的Hyperledger Fabric区块链网络执行交易，并生成/运行与区块链网络交互的示例Angular 2应用程序。

本教程给出了对该技术的概述，以及可用于你自己用例的可用资源。

注意：本教程是针对在Ubuntu Linux上构建的最新的Hyperledger Composer编写的，它与Hyperledger Fabric v1.0运行在一起，也针对Mac环境进行了测试。

### 先决条件
开始本教程之前：

 - [建立你的开发环境](https://hyperledger.github.io/composer/installing/development-tools.html)
 - 安装一个编辑器，例如VSCode或Atom
### 第一步：创建一个商业网络结构
关键概念是**商业网络定义(BND)**。它为你的区块链解决方案定义了数据模型、交易逻辑和访问控制规则。要创建一个BND，我们需要在磁盘上创建一个合适的项目结构。

最简单的方法就是使用Yeoman生成器创建骨架商业网络。这将创建一个包含商业网络的所有组件的目录。

1. 使用Yeoman创建一个骨架商业网络。此命令将需要一个商业网络名称、说明、作者姓名、作者电子邮件地址、许可证选择和命名空间。
```bash
yo hyperledger-composer:businessnetwork
```
2. 输入`tutorial-network`网络名称，以及描述，作者姓名和作者电子邮件的所需信息。

3. 选择`Apache-2.0`作为许可证。

4. 选择`org.acme.biznet`作为命名空间。

### 第二步：定义一个商业网络
商业网络由资产、参与者、交易、访问控制规则以及可选的事件和查询组成。在前面步骤中创建的骨架商业网络中，有一个模型(`.cto`)文件，其中将包含商业网络中所有资产、参与者和交易的类定义。骨架商业网络还包含了一个具有基本访问控制规则的访问控制（`permissions.acl`）文档，一个含有交易处理器函数的脚本（`logic.js`）文件以及一个包含商业网络元数据的`package.json`文件。

#### 对资产、参与者和交易建模
第一个要更新的文档是模型l（`.cto`）文件。该文件使用Hyperledger Composer建模语言编写。这个模型文件包含资产、交易、参与者和事件的每个类别的定义。它隐含地扩展了建模语言文档中描述的Hyperledger Composer系统模型。

1. 打开`org.acme.biznet.cto`模型文件。

2. 替换为以下内容：
```
/**
 * My commodity trading network
 */
namespace org.acme.biznet
asset Commodity identified by tradingSymbol {
    o String tradingSymbol
    o String description
    o String mainExchange
    o Double quantity
    --> Trader owner
}
participant Trader identified by tradeId {
    o String tradeId
    o String firstName
    o String lastName
}
transaction Trade {
    --> Commodity commodity
    --> Trader newOwner
}
```
3. 保存你的更改到`org.acme.biznet.cto`。

#### 添加JavaScript交易逻辑
在模型文件中，定义了一个`Trade`交易，指定与资产的关系和一个参与者。交易处理器函数文件包含执行模型文件中定义的交易的JavaScript逻辑。

这个`Trade`交易被设定为简单地接收要交易的类型为`Commodity`的资产，以及类型为`Trader`的参与者作为新的拥有者。

 1. 打开`logic.js`脚本文件。

 2. 替换为以下内容：
```
/**
 * Track the trade of a commodity from one trader to another
 * @param {org.acme.biznet.Trade} trade - the trade to be processed
 * @transaction
 */
function tradeCommodity(trade) {
    trade.commodity.owner = trade.newOwner;
    return getAssetRegistry('org.acme.biznet.Commodity')
        .then(function (assetRegistry) {
            return assetRegistry.update(trade.commodity);
        });
}
```
保存你的更改到`logic.js`。

#### 添加访问控制
1. 在`tutorial-network`目录中创建一个`permissions.acl`文件。

2. 将以下访问控制规则添加到`permissions.acl`：
```
/**
 * Access control rules for tutorial-network
 */
rule Default {
    description: "Allow all participants access to all resources"
    participant: "ANY"
    operation: ALL
    resource: "org.acme.biznet.*"
    action: ALLOW
}

rule SystemACL {
  description:  "System ACL to permit all access"
  participant: "ANY"
  operation: ALL
  resource: "org.hyperledger.composer.system.**"
  action: ALLOW
}
```
保存你的更改到`permissions.acl`。

### 第三步：生成一个商业网络档案
现在已经定义了商业网络，它必须打包到可部署的商业网络档案（.bna）文件中。

1. 使用命令行，导航到`tutorial-network`目录。

2. 从`tutorial-network`目录中运行以下命令：
```bash
composer archive create -t dir -n .
```
在命令运行之后，一个叫`tutorial-network@0.0.1.bna`的商业网络存档文件在`tutorial-network`目录中被创建。

### 第四步：部署商业网络
创建`.bna`文件后，商业网络就可以部署到Hyperledger Fabric实例。通常情况下，为了创建一个`PeerAdmin`身份，需要来自Fabric管理员的信息，还需要有将链码部署到peer的权限。但是，作为开发环境安装的一部分，已经创建了一个`PeerAdmin`身份。

在运行时被安装后，就可以将一个商业网络部署到peer。根据最佳实践，应该创建一个新的身份来管理部署后的商业网络。这个身份被称为网络管理员。

#### 获取正确的凭据
一个有正确凭据的`PeerAdmin`商业网络卡片已作为开发环境安装的一部分而创建。

#### 部署商业网络
为了将商业网络部署到Hyperledger Fabric，需要在peer上安装Hyperledger Composer链码，然后必须将商业网络档案（`.bna`）发送给peer，并且必须创建一个新的参与者（由卡片标识和关联）作为网络管理员。最后，必须导入网络管理员商业网络卡片，才能ping通网络，查看是否有响应。

1. 要安装Composer运行时，请运行以下命令：
```bash
composer runtime install --card PeerAdmin@hlfv1 --businessNetworkName tutorial-network
```
该`composer runtime install`命令需要一个`PeerAdmin`商业网络卡片（在这案例中已经预先创建和导入了一个）以及商业网络的名称。

2. 为了部署商业网络，要从目录`tutorial-network`中运行以下命令：
```bash
composer network start --card PeerAdmin@hlfv1 --networkAdmin admin --networkAdminEnrollSecret adminpw --archiveFile tutorial-network@0.0.1.bna --file networkadmin.card
```
该`composer network start`命令需要一个商业网络卡片，以及商业网络的管理员身份名称、`.bna`文件路径和要创建的文件的名称（即准备导入的商业网络卡片）。

3. 要将网络管理员身份导入为可用的商业网络卡片，请运行以下命令：
```bash
composer card import --file networkadmin.card
```
该`composer card import`命令需要指定一个文件名，就是在`composer network start`命令创建卡片时指定的。

4. 要检查商业网络是否已成功部署，请运行以下命令来ping网络：
```bash
composer network ping --card admin@tutorial-network
```
该`composer network ping`命令需要一张商业网络卡片来认证要ping的网络。

### 第五步：生成一个REST服务器
Hyperledger Composer可以基于商业网络生成定制的REST API。为了开发Web应用程序，REST API提供了一个与语言无关的有用抽象层。

1. 要创建REST API，请导航到`tutorial-network目`录并运行以下命令：
```bash
composer-rest-server
```
2. 输入`admin@tutorial-network`作为卡片名称。

3. 当询问是否在生成的API中使用名称空间时，请选择**不使用名n称空间**。

4. 当询问是否保护生成的API时选择**否**。

5. 当询问是否启用事件发布时，选择**是**。

6. 当询问是否启用TLS安全时，请选择**否**。

生成的API连接到部署的区块链和商业网络。
