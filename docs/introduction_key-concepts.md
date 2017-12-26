# Hyperledger Composer的关键概念

Hyperledger Composer是一种编程模型，包含建模语言和一组API，可快速定义和部署业务网络和应用程序，允许**参与者**发送交换**资产的****交易**。

## Hyperledger Composer组件

你可以使用我们基于浏览器的界面（称为Hyperledger Composer Playground）体验Hyperledger Composer。Playground可作为托管版本（无需安装）或本地安装（适用于离线编辑和测试示例业务网络）。

想要使用Hyperledger Composer的完整应用程序开发功能的开发人员应安装开发人员工具。

![Hyperledger Composer组件图](https://hyperledger.github.io/composer/assets/img/ComposerComponents.svg)

## HyperledgerComposer的关键概念

### 区块链状态存储

通过业务网络提交的所有交易都存储在区块链账本中，资产和参与者的当前状态存储在区块链状态数据库中。区块链将账本和状态数据库分布在一组peer上，并确保使用一致性算法在所有peer上对账本和状态数据库的更新是一致的。

### 连接配置文件

Hyperledger Composer使用*连接配置文件*连接到运行时。连接配置文件是一个JSON文档，位于用户的主目录（或可能来自环境变量），并在使用Composer API或命令行工具时用名称引用。使用连接配置文件可确保代码和脚本轻松地从一个运行时实例移植到另一个实例。你可以在参考部分阅读更多关于连接配置文件的文档。

### 资产

资产是有形或无形的商品、服务或财产，并存储在库(registry)中。资产几乎可以代表业务网络中的任何东西，例如，待售房屋、销售清单、该房屋的土地登记证书以及该房屋的保险文件都可以是一个或多个业务网络中的资产。

资产必须具有唯一的标识符(identifier)，但除此之外，它们可以包含你定义的任何属性。资产可能*与*其他资产或参与者关联。

### 参与者

参与者是业务网络的成员。他们可能拥有资产并提交交易。像资产一样，参与者类型被建模为必须具有标识符并且可以具有任何其他属性。

### 身份和身份卡片

在业务网络中，参与者可以与身份相关联。身份卡片是身份、连接配置文件和元数据的组合。身份卡片简化了连接到业务网络的过程，并将业务网络之外的身份概念扩展到身份“钱包”，每个身份与特定的业务网络和连接配置文件相关联。

### 交易

交易是参与者与资产交互的机制。这可以像参与者对拍卖中的资产进行投标或者标记拍卖的拍卖人一样简单，自动将资产的所有权转移给出价最高的投标者。

### 查询

查询用于返回有关区块链世界状态的数据。查询在业务网络中定义，并且可以包含用于简单定制的变量参数。通过使用查询，可以轻松地从你的区块链网络中提取数据。查询使用Hyperledger Composer API发送。

### 事件

业务网络定义中的事件与资产或参与者相同。一旦事件被定义，它们就可以被交易处理器函数发出，以向外部系统指出账本发生了重要的事情。应用程序可以通过`composer-client`API 订阅发出的事件。

### 访问控制

业务网络可能包含一组访问控制规则。访问控制规则允许对参与者访问业务网络中的哪些资产以及在什么条件下访问进行细粒度的控制。访问控制语言足够丰富，可以通过声明捕捉复杂的条件，例如“只有车主才能转让车辆所有权”。交易处理器函数逻辑的外部访问控制使检查、调试、开发和维护变得更加容易。

### 历史库

历史库是一个专门的记录成功交易的库，包括提交他们的参与者和身份。历史库将交易存储为在Hyperledger Composer系统名称空间中定义的`HistorianRecord`资产。

## 我从哪里出发？

- 要立即尝试Hyperledger Composer，请参阅[在线Playground](https://hyperledger.github.io/composer/installing/getting-started-with-playground.html)
- 有关使用Composer构建的典型解决方案的架构概述，请参阅“ [典型解决方案架构”](https://hyperledger.github.io/composer/introduction/solution-architecture.html)
- 要安装开发工具，请参阅[使用开发工具进行安装](https://hyperledger.github.io/composer/installing/development-tools.html)
