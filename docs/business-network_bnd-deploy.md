# 部署业务网络

在部署业务网络定义之前，必须将其打包到*业务网络档案*（.bna）文件中。`composer archive create`命令用于从磁盘上的业务网络定义文件夹创建业务网络档案文件。

一旦创建了业务网络档案文件，就可以使用[`composer runtime install`](https://hyperledger.github.io/composer/reference/composer.runtime.install.html)命令和[`composer network start`](https://hyperledger.github.io/composer/reference/composer.network.start.html)命令将其部署到运行时。

例如：
```bash
composer runtime install -n tutorial-network -c PeerAdmin@fabric-network
```

要更新已部署业务网络的业务网络定义，请使用[`composer network update`](https://hyperledger.github.io/composer/reference/composer.network.update.html)CLI命令。

## 将业务网络部署到Hyperledger Fabric v1.0

在Hyperledger Fabric v1.0中，peer强制使用管理员和成员（或用户）的概念。管理员有权将新建业务网络的Hyperledger Fabric链码安装到peer。成员没有安装链码的权限。为了将业务网络部署到一组peer，你必须提供具有所有这些peer的管理权限的身份。

要使该身份及其证书可用，你必须使用与peer管理员身份关联的证书和私钥创建peer管理业务网络卡片。Hyperledger Composer提供了一个示例Hyperledger Fabric v1.0网络。此网络的peer管理员被称为`PeerAdmin`，并且使用示例脚本启动网络时会自动为你导入该身份。请注意，对于其他Hyperledger Fabric v1.0网络，peer管理员可能会被赋予不同的名称。

## 业务网络管理员

在部署业务网络时，根据业务网络定义中指定的访问控制规则强制执行访问控制。每个业务网络必须至少有一个参与者，并且该参与者必须具有访问该业务网络的有效身份。否则，客户端应用程序将无法与业务网络进行交互。

业务网络管理员是业务网络部署后负责为其组织配置业务网络的参与者，并负责载入组织的其他参与者。由于业务网络包括多个组织，因此对于任何给定的业务网络应该有多个业务网络管理员。

Hyperledger Composer提供了一个内置的参与者类型，`org.hyperledger.composer.system.NetworkAdmin`，代表了业务网络管理员。此内置参与者类型没有任何特殊权限; 它们仍然服从业务网络定义中指定的访问控制规则。因此，建议你从以下示例访问控制规则开始，向业务网络管理员授予对业务网络的完全访问权限：
```
rule NetworkAdminUser {
    description: "Grant business network administrators full access to user resources"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: ALL
    resource: "**"
    action: ALLOW
}

rule NetworkAdminSystem {
    description: "Grant business network administrators full access to system resources"
    participant: "org.hyperledger.composer.system.NetworkAdmin"
    operation: ALL
    resource: "org.hyperledger.composer.system.**"
    action: ALLOW
}
```

默认情况下，Hyperledger Composer将在部署期间自动创建一个业务网络管理员参与者。用于部署业务网络的身份也将绑定到该业务网络管理员参与者，以便在部署之后可以使用身份与业务网络进行交互。

Hyperledger Fabric peer管理员可能没有权限使用Hyperledger Fabric证书颁发机构（CA）颁发新身份。这可能会限制业务网络管理员加入其他组织的其他参与者的能力。因此，最好创建一个业务网络管理员，让该管理员具有使用Hyperledger Fabric Certificate Authority（CA）颁发新身份的权限。

你可以使用其他选项来[`composer network start`](https://hyperledger.github.io/composer/reference/composer.network.start.html)指定在部署业务网络期间应创建的业务网络管理员。

如果业务网络管理员具有登记ID和登记密码，则可以使用`-A`（业务网络管理员）和`-S`（业务网络管理员用的登记密码）标志。例如，以下命令将为现有`admin`登记ID 创建一个业务网络管理员：
```bash
composer network start -c PeerAdmin@fabric-network -A admin -S
```

## 在本地使用Playground部署业务网络

在本地使用Playground将业务网络部署到Hyperledger Fabric v1.0时，必须按照上述过程使用peer管理员身份进行连接。

Playground中的身份与业务网络卡片相关联，包括连接配置文件、身份元数据和证书。

在本地部署使用Playground的业务网络时，必须至少有一个具有`PeerAdmin`角色的业务网络卡片和至少一个具有`ChannelAdmin`角色的业务网络卡片。每个这些业务网络卡片都必须包含正确的管理员证书。

## 使用Hyperledger Composer Playground将业务网络部署到本地Fabric碰到的问题

使用本地安装的Hyperledger Composer Playground将业务网络部署到Hyperledger Fabric实例时，可能会遇到以下错误：
```
Error: error trying to list instantiated chaincodes. Error: chaincode error (status 500, message: Authorization for GETCHAINCODES on channel getchaincodes has been denied with error Failed verifying that proposal's creator satisfies local MSP principal during channelless check policy with policy [Admins]:[This identity is not an admin])
```

一旦发生此错误，你必须删除本地浏览器存储以恢复正常功能。*请注意*：删除本地浏览器存储将删除你的连接配置文件和你的钱包中的身份。有关此错误的更多信息，请参阅[特定的错误页面](https://hyperledger.github.io/composer/problems/deployment-local-playground.html)

## 参考

- [**Composer CLI命令**](https://hyperledger.github.io/composer/reference/commands.html)
