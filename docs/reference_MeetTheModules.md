# Hyperledger Composer npm模块

Hyperledger Composer为应用程序开发人员提供了3个主要模块。如果你正在编写一个应用程序，这是你的入口点。

1. `composer-client`

2. `composer-admin`

3. `composer-cli`

`composer-client`和`omposer-admin`是为应用程序提供API的两个模块。node.js应用程序应该只使用来自这些模块的API。如果还有其他需要的API，请联系我们。

所有的API的细节已经记录在JSDocs中（参见参考资料）。

## composer-client

这个模块通常作为应用程序的本地依赖来安装。它提供了业务应用程序用来连接到业务网络以访问**资产**、**参与者**和提交**交易**的API 。在生产中，这是需要添加为应用程序的直接依赖的唯一模块。
```bash
npm install --save composer-client
```

## composer-admin

通常将该模块安装为**管理**应用程序的本地依赖项。该API允许创建和部署**业务网络定义**。
```bash
npm install --save composer-admin
```

## composer-cli

这个命令行工具提供了部署和管理业务网络定义的能力。这通常会被安装为一个全局模块。
```bash
npm install -g composer-cli
```

如果你希望，可以把它当作本地依赖，但是你可能需要直接访问cli.js节点类而不是使用`composer`命令。
