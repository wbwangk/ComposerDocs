# 参与者和身份

## 概念

A `Participant`(参与者)是业务网络中的行为人(actor)。参与者可能是一个组织的分支。参与者可以创建资产，并与其他参与者交换资产。参与者通过提交交易来处理资产。

参与者拥有一组`Identity`文档，可以用来证明参与者的身份。例如，一个人可能有一个或多个以下身份文档证明他们是谁：

- 护照

- 驾驶执照

- 指纹

- 视网膜扫描

- SSL证书

在Hyperledger Composer中，参与者可以通过一组身份文档（用来与业务网络进行交互）来区分。

为了使新参与者加入业务网络，必须在业务网络中创建该参与者的新实例。参与者实例存储关于该参与者的所有必需信息，但它不赋予该参与者与业务网络交互的访问权限。

为了授予参与者与业务网络进行交互的权限，身份文档必须被颁发（`Issued`）给该参与者。新的参与者然后可以使用该身份文档与业务网络进行交互。

参与者可能拥有一个现有的身份文档，用于与其他业务网络或其他外部系统进行交互。这些身份文档可以被重用和绑定（`Bound`）到该参与者。新的参与者然后可以使用他们现有的身份文档与业务网络进行交互。

身份文档通常在一段时间后过期。身份文档也可能丢失或被盗。如果身份文档过期，或者需要更换，则必须撤销(`Revoked`）使其不能再用于与业务网络进行交互。

但是，撤销身份文档并不会删除有关该参与者及其拥有的任何资产的信息。撤销身份文档简单地删除参与者使用该身份文档与业务网络交互的能力。可以通过向参与者颁发新的身份文档来恢复对业务网络的访问。

这些对参与者和身份的管理行为由业务网络中的现有参与者执行，例如监管机构或者同一组织中被信任来管理该组织中的参与者/身份的参与者。

## Hyperledger Composer中的参与者和身份

在Hyperledger Composer中，参与者的结构在模型文件中建模。该结构可以包括关于参与者的各种信息，例如参与者姓名、地址、电子邮件地址、出生日期等。该被建模参与者的新实例然后会被创建和添加到参与者库中。

Hyperledger Composer需要使用区块链身份作为身份文档（存放）的方式。例如，在将业务网络部署到Hyperledger Fabric时，将使用登记证书作为身份文档的格式。这些登记证书用于对提交到已部署业务网络的交易进行数字签名。

已部署的业务网络在身份库（`Identity Registry`）中维护一组身份映射到参与者。当一个身份被颁发（`Issued`）或绑定（`Bound`）某参与者，一个新的映射被添加到身份库中。当该参与者使用该身份将交易提交到已部署的业务网络时，Composer运行时会在身份库中查找该身份的有效映射。这种查找是使用公钥签名或指纹完成的，指纹本质上是证书内容（对证书和身份是唯一的）的散列。

在身份库中找到映射后，将从该映射中检索该身份的参与者。参与者成为当前参与者（`Current Participant`），即提交交易的参与者。Hyperledger Composer中的所有访问控制都基于当前参与者。访问控制规则定义哪些参与者可以执行哪些操作，哪些资源在当前参与者上操作。

当参与者第一次使用身份向已部署的业务网络提交事务时，该身份是活动的（`Activated`）。这意味着身份库中的条目被更新，以记录第一次使用身份的事实。有关身份的其他信息（例如证书）也可能在激活期间记录在身份库中，如果该信息在身份被颁发或绑定到参与者时不可用。

如果当一个身份被撤销时，身份库中的该身份的条目被更新以将状态更改为已撤销（`Revoked`）。身份被撤销后，如果参与者尝试使用该身份向已部署的业务网络提交事务，则该事务会被拒绝。

## Hyperledger Composer Playground中的身份和ID

在Hyperledger Composer Playground中，有一个包含本地存储的身份卡片的钱包。身份卡片是企业网络的访问卡片，包括身份数据，连接配置文件以及用于业务网络访问的正确证书。可以导出身份卡片以允许将身份分配给他人。

## 在Hyperledger Composer中执行身份管理任务

Hyperledger Composer Node.js客户端API、REST API和命令行接口都可以用来执行身份管理操作。例如，以下身份管理操作可通过所有(三种)Hyperledger Composer接口使用：

- 将新参与者添加到参与者库

- 向参与者颁发新的身份

- 将现有身份绑定到参与者

- 撤销参与者的身份

- 列出已部署的业务网络中的所有身份

有关更多信息，请参阅本文档底部的相关任务和参考资料。

## 相关的概念

[业务网络](business-network_business-network-index.html)

## 相关任务

[创建业务网络定义](business-network_bnd-create.md)  
[添加参与者](managing_participant-add.md)  
[向参与者颁发新的身份](managing_identity-issue.md)  
[将现有身份绑定到参与者](managing_identity-bind.md)  
[列出业务网络中中所有身份](managing_identity-list.md)  
[从参与者中撤消身份](managing_identity-revoke.md)  

## 相关参考

[composer参与者添加](https://hyperledger.github.io/composer/stable/reference/composer.participant.add.html)  
[composer身份颁发](https://hyperledger.github.io/composer/stable/reference/composer.identity.issue.html)  
[composer身份绑定](https://hyperledger.github.io/composer/stable/reference/composer.identity.bind.html)  
[composer身份撤销](https://hyperledger.github.io/composer/stable/reference/composer.identity.revoke.html)  
[composer身份列表](https://hyperledger.github.io/composer/stable/reference/composer.identity.list.html)  
 
