# 典型的Hyperledger Composer解决方案架构

Hyperledger Composer使架构师和开发人员能够快速创建“全堆栈”区块链解决方案。即业务逻辑运行在区块链上运行，REST API将区块链逻辑暴露给Web或移动应用程序，以及将区块链与现有企业记录系统集成在一起。

![典型的Hyperledger Composer体系结构图](https://hyperledger.github.io/composer/assets/img/ComposerArchitecture.svg)

Hyperledger Composer由以下高级组件组成：

- 执行运行时（目前支持四个！）
- JavaScript SDK
- 命令行接口
- REST服务器
- LoopBack连接器
- PlaygroundWeb用户界面
- Yeoman代码生成器
- VSCode和Atom编辑器插件

## 执行运行时

Hyperledger Composer被设计为支持不同的可插入运行时，目前有三个运行时实现：
-  Hyperledger Fabric版本1.0。状态存储在分布式账本上。
-  Web，在网页中执行，由Playground使用。状态存储在浏览器本地存储中。
-  Embedded，在Node.js进程中执行，主要用于Fabric单元测试。状态存储在内存中的键值存储中。

### 连接配置文件

Hyperledger Composer使用连接配置文件来确定如何连接到执行运行时。每种*类型*的执行运行时都有不同的配置选项。例如，Hyperledger Fabric版本1.0运行时的连接配置文件将包含Fabric peer的TCP/IP地址和端口以及加密证书等。

连接配置文件按名称（在代码和命令行中）引用，连接配置文件（以JSON格式）从用户的主目录中解析。

## JavaScript SDK

Hyperledger Composer JavaScript SDK是一组Node.js API，使开发人员能够创建应用程序来管理和与已部署业务网络进行交互。

这些API分成两个npm模块：

1. `composer-client` 用于向业务网络提交交易或对资产和参与者执行创建、读取、更新和删除操作
2. `composer-admin` 用于管理业务网络（部署、取消部署）

所有API的细节都可以作为JSDocs（请参阅参考资料）。

### Composer客户端

这个模块通常作为应用程序的本地依赖来安装。它提供了业务应用程序用来连接到业务网络以访问**资产**、**参与者**和提交**交易**的API 。在生产中，这是需要添加为应用程序的直接依赖的唯一模块。

### Composer管理员

通常将该模块安装为**管理**应用程序的本地依赖项。该API允许创建和部署**业务网络定义**。

## 命令行接口

该`composer`命令行工具使开发人员和管理员部署和管理业务网络定义。

## REST服务器

Hyperledger Composer REST服务器自动为业务网络生成Open API（Swagger）REST API。REST服务器（基于LoopBack技术）将业务网络的Composer模型转换为Open API定义，并在运行时实现对资产和参与者的创建、读取、更新和删除支持，并允许交易提交处理或获取。

## LoopBack连接器

Hyperledger Composer LoopBack连接器由Composer REST服务器使用，但也可以由本机支持LoopBack的集成工具单独使用。或者，它可以与LoopBack工具一起使用，以创建更复杂的REST API自定义。

## Playground Web用户界面

Hyperledger Composer Playground是定义和测试业务网络的Web用户界面。它允许业务分析师快速导入在Web或Hyperledger Fabric运行时执行的范例和原型业务逻辑。

## Yeoman代码生成器

Hyperledger Composer使用Open Source Yeoman代码生成器框架来创建框架项目：

- Angular Web应用程序
- Node.js应用程序
- 骨架的业务网络

## VSCode和Atom编辑器扩展

Hyperledger Composer为VSCode和Atom提供了社区贡献的编辑器扩展。VSCode扩展非常强大，验证了Composer模型和ACL文件，提供语法高亮显示、错误检测和片段支持。Atom插件要简单得多，只有基本的语法高亮。
