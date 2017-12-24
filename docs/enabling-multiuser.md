# 为REST服务器启用多用户模式

默认情况下，Hyperledger Composer REST服务器使用启动时在命令行上指定的区块链身份服务所有请求。例如，在使用以下命令时，对REST服务器发出的所有请求都将使用区块链身份**alice1**来服务，并对所有交易进行数字签名:
```bash
composer-rest-server -c alice1@my-network
```

这意味着业务网络无法区分REST服务器的不同客户端。在某些使用情况下，这可能是可以接受的，例如，如果区块链身份只有只读访问权限，而REST服务器使用API管理网关进行安全保护。

REST服务器可以配置为多用户模式。多用户模式允许REST服务器的客户端为数字签名交易提供他们自己的区块链身份。这使业务网络能够区分REST服务器的不同客户端。

多用户模式要求启用[REST API身份认证](https://hyperledger.github.io/composer/stable/integrating/enabling-rest-authentication.html)，并且如果未明确指定，将自动启用REST API身份认证。你必须选择并配置Passport策略才能对用户进行身份认证。REST API身份认证是必需的，以便可以识别客户端。

一旦客户端已经对REST API进行了认证，该客户端就可以将区块链身份添加到钱包中。该钱包对于该客户端是私密的，并且不能被其他客户端访问。当客户端向REST服务器发出请求时，客户端钱包中的区块链身份用于对该客户端所做的所有交易进行数字签名。

请注意，此功能要求客户端信任REST服务器。此信任是必需的，因为此功能要求REST服务器存储客户端区块链身份（包括私钥）。因此，强烈建议客户端只使用由受信任方（例如其组织内的管理员）管理的REST服务器。

## 启动启用了多用户模式的REST服务器

继续之前，你必须配置环境变量`COMPOSER_PROVIDERS`。有关如何执行此任务的说明，请在继续之前阅读以下主题:[启用REST服务器的身份认证](https://hyperledger.github.io/composer/stable/integrating/enabling-rest-authentication.html)

你可以使用`-m true`参数在启动启用了多用户模式的REST服务器。一旦启用了多用户模式，客户端在向业务网络发出任何请求之前都必须进行身份认证。

例如，以下是作为开发者教程一部分部署的业务网络的命令，但是你可能需要修改你的业务网络的命令:
```bash
composer-rest-server -c admin@my-network -m true
```

该`-m true`参数自动启用了REST API身份认证。如果你希望明确的话，你也可以提供两个参数，`-a true -m true`。在继续之前，你必须使用身份认证已配置机制对REST API进行身份认证。

现在，导航到[http:// localhost:3000/explorer/](http://localhost:3000/explorer/)上的REST API浏览器。如果已成功启用多用户模式，则任何使用REST API浏览器调用业务网络REST API操作的任何尝试都会被拒绝，并显示一条`A business network card has not been specified`错误消息。

如果你看到`HTTP 401 Authorization Required`错误消息，则说明你没有正确地对REST API进行身份认证。

## 将业务网络卡片添加到钱包

首先，你必须向业务网络中的参与者发放区块链身份。这个例子将假设你已经向`alice1`参与者发放了区块链身份`org.acme.mynetwork.Trader#alice@email.com`，并且你为这个区块链身份创建了一个业务网络卡片，卡片存储在文件`alice1@my-network.card`中。

请按照以下步骤将业务网络卡片添加到钱包:

1. 导航到[http://localhost:3000/explorer/](http://localhost:3000/explorer/)上的REST API浏览器，然后通过展开**钱包**类别导航到钱包API 。

2. 通过调用`GET/wallet`操作检查钱包是否包含任何业务网络卡片。操作的回应应该是:
   ```
   []
   ```

3. 通过调用`POST /wallet/import`操作将业务网络卡片导入钱包。你必须通过单击**选择文件**按钮来指定业务网络卡片文件`alice1@my-network.card`。操作的回应应该是:
   ```
   no content
   ```

   业务网络卡片`alice1@my-network`现在已经被导入到钱包中。

4. 通过调用`GET /wallet`操作检查钱包是否包含业务网络卡片`alice1@my-network`。操作的回应应该是:
   ```json
   [
       {
           "name": "alice1@my-network",
           "default": true
       }
   ]
   ```

   业务网络卡片`alice1@my-network`被显示出来。`default`属性的值是`true`，这意味着当与业务网络交互时，该业务网络卡片将被默认使用。

现在，导航到[http://localhost:3000/explorer/](http://localhost:3000/explorer/)上的REST API浏览器。尝试再次使用REST API浏览器调用其中一个业务网络REST API操作。这一次，调用应该成功。

你可以通过调用`GET /system/ping`操作来测试区块链身份。此操作将返回区块链身份发给的参与者的完全限定标识符:
```
{
  "version": "0.8.0",
  "participant": "org.acme.mynetwork.Trader#alice@email.com"
}
```

## 最后的笔记

当REST服务器在启用多用户模式的情况下启动时，客户端所做的所有REST API请求都使用存储在客户端钱包中的区块链身份。启动时在命令行上指定的区块身份不用于处理任何请求; 它仅用于初始连接到业务网络并下载生成REST API所需的业务网络定义。因此，在命令行中指定的区块链身份只需要最小的权限即连接能力和下载业务网络定义的能力，而不需要任何资产、参与者或交易的许可。

所有用户信息通过使用LoopBack连接器保留在LoopBack数据源中。默认情况下，REST服务器使用LoopBack“内存”连接器来保存用户信息，当REST服务器终止时会丢失这些信息。REST服务器应配置一个LoopBack连接器，用于将数据存储在高度可用的数据源（例如数据库）中。有关更多信息，请参阅[部署REST服务器](https://hyperledger.github.io/composer/stable/integrating/deploying-the-rest-server.html)。
