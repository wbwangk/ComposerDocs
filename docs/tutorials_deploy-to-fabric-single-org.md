## 为单个组织部署Hyperledger Composer区块链网络到Hyperledger Fabric
[原文](https://hyperledger.github.io/composer/unstable/tutorials/deploy-to-fabric-single-org)

在[开发环境](installing_development-tools.md)中，为你创建了一个简单Hyperledger Fabric网络(`fabric-dev-servers`)，以及为部署区块链业务网络所需的所有Hyperledger Composer配置。

本教程将演示为单个组织将区块链业务网络部署到的Hyperledger Fabric实例，管理员需要执行的步骤，包括如何生成必需的Hyperledger Composer配置。随后的教程将演示如何将区块链业务网络部署到多个组织的Hyperledger Fabric实例。

### 先决条件
 1. 在你继续前，确保你已经完成了[安装一个开发环境](installing_development-tools.md)中步骤。  

### 步骤一：启动Hyperledger Fabric网络
为了遵循本教程，必须必须启动Hyperledger Fabric网络。你可以使用开发环境中提供的简单Hyperledger Fabric网络，也可以使用你根据Hyperledger Fabric文档构建的自己的Hyperledger Fabric网络。

本教程将假设你使用开发环境中提供的简单Hyperledger Fabric网络。如果你使用自己的Hyperledger Fabric网络，则必须在下面详细介绍的配置和你自己的配置之间进行映射。

 1. 通过运行以下命令启动一个干净的Hyperledger Fabric：
```bash
cd ~/fabric-tools
./stopFabric.sh
./teardownFabric.sh
./downloadFabric.sh
export FABRIC_VERSION=hlfv11        
./startFabric.sh
```
 2. 删除你的钱包中的所有业务网络卡片。忽略无法找到业务网络卡片的错误，这是安全的：
```
composer card delete -n PeerAdmin@fabric-network
composer card delete -n admin@tutorial-network
```

### 步骤二：探索Hyperledger Fabric网络
这一步将探索刚刚启动的Hyperledger Fabric网络，以便了解它是如何配置的，以及它由哪些组件组成。在后续步骤中配置Hyperledger Composer时，你将使用本节中的所有信息。

#### 配置文件
在开发环境中提供的简单Hyperledger Fabric网络，已经使用Hyperledger Fabric配置工具`cryptogen`和`configtxgen`来配置。

`cryptogen`的配置存储在下面的文件中：
```
~/fabric-tools/fabric-scripts/hlfv11/composer/crypto-config.yaml
```
`configtxgen`的配置存储在下面的文件中：
```
~/fabric-tools/fabric-scripts/hlfv11/composer/configtx.yaml
```
你可以通过阅读Hyperledger Fabric文档来找到有关这些配置工具的更多信息，它们做什么和如何使用它们。

#### 组织
简单的Hyperledger Fabric网络由一个叫做`Org1`的组织构成。该组织使用域名`org1.example.com`。此外，该组织的成员服务提供商(MSP)被称为`Org1MSP`。在本教程中，你将部署只有组织`Org1`才能与之交互的区块链业务网络。

#### 网络组件
Hyperledger Fabric网络由多个组件组成：  
- 一个Org1的peer节点，名叫peer0.org1.example.com  
  - 请求端口是7051  
  - 事件hub端口是7053  
- 一个Org1的CA(Certificate Authority)，名叫ca.org1.example.com  
  - CA端口是7054  
- 一个排序节点，名叫orderer.example.com  
  - 排序端口是7050  

Hyperledger Fabric网络组件在Docker容器中运行。在Docker容器中运行Hyperledger Composer时，可以使用上面的名称（例如，`peer0.org1.example.com`）与Hyperledger Fabric网络进行交互。

本教程将在Docker主机上运行Hyperledger Composer命令，而不是在Docker网络内运行。这意味着Hyperledger Composer命令必须使用`localhost`主机名和暴露的容器端口与Hyperledger Fabric网络进行交互。

#### 用户
组织`Org1`使用名为`Admin@org1.example.com`的用户进行配置。此用户是管理员。组织的管理员有权将区块链业务网络的代码安装到其组织的peer，并且还可以具有启动区块链业务网络的权限，具体取决于配置。在本教程中，你将以`Admin@org1.example.com`用户身份部署区块链业务网络。

用户`Admin@org1.example.com`有一组存储在下列目录中的证书和私钥文件：
```
~/fabric-tools/fabric-scripts/hlfv11/composer/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
```
你稍后将使用其中一些文件与Hyperledger Fabric网络进行交互。

除了管理员之外，`Org1`的CA（证书颁发机构）使用一个默认用户进行配置。此默认用户登记ID是`admin`，登记密码是`adminpw`。但是，这个用户无权部署区块链业务网络。

#### 信道
最后，创建了一个名为`channel`的信道。peer节点`peer0.org1.example.com`已加入此通道。你只能将Hyperledger Composer区块链业务网络部署到已存在的信道，但您可以按照Hyperledger Fabric文档创建其他信道。

### 步骤三：建立连接配置文件
连接配置文件(profile)指定了定位和连接Hyperledger Fabric网络所需的所有信息，例如所有Hyperledger Fabric网络组件的主机名和端口。在此步骤中，你将为Hyperledger Composer创建一个连接配置文件，用于连接到Hyperledger Fabric网络。

 1. 创建一个名为的`connection.json`的连接配置文件。

 2. 通过将以下三行添加到`connection.json`顶部来提供连接配置文件的`name`和`type`属性：
```json
{
  "name": "fabric-network",
  "type": "hlfv1",
```
连接配置文件中的`name`属性给出Hyperledger Fabric网络的名称，所以稍后可以引用它。在刚创建的连接配置文件，名称是`fabric-network`。你可以使用喜欢的名字为Hyperledger Fabric网络命名。

Hyperledger Composer被设计为兼容不同类型的区块链网络。目前，仅支持Hyperledger Fabric v1.0，但你必须指定要使用的区块链网络的类型。Hyperledger Fabric v1.0的类型是`hlfv1`。

 3. 必须指定用于连接到Hyperledger Fabric网络的MSP的名称：
```json
"mspID": "Org1MSP",
```
我们正在以`Org1`连接，`Org1`的MSP被称为`Org1MSP`。

我们必须指定要连接的Hyperledger Fabric网络中所有peer节点的主机名和端口。
```json
"peers": [
    {
        "requestURL": "grpc://localhost:7051",
        "eventURL": "grpc://localhost:7053"
    }
],
```
在这里，我们已经指定了单个peer节点`peer0.org1.example.com`（使用主机名`localhost`），请求端口7051和事件hub端口7053。

`peers`数组可以包含多个peer节点。如果您有多个peer节点，则应将其全部添加到peers数组中，以便Hyperledger Composer可以与它们进行交互。

区块链业务网络将部署到所有指定的peer节点。一旦区块链业务网络部署完毕，指定的peer节点将用于查询区块链业务网络、为交易背书和订阅事件。

 5. 我们必须在Hyperledger Fabric网络中指定证书颁发机构（CA）的主机名和端口，用于登记现有用户和注册新用户。
```json
"ca": {
    "url": "http://localhost:7054",
    "name": "ca.org1.example.com"
},
```
这里我们指定了一个CA`ca.or1.example.com`（使用主机名`localhost`）和CA端口7054。

 6. 我们必须指定要连接的Hyperledger Fabric中所有排序节点的主机名和端口。
```json
"orderers": [
    {
        "url" : "grpc://localhost:7050"
    }
],
```
在这里，我们已经指定了一个排序节点`orderer.example.com`（使用主机名`localhost`）和排序端口7050。

该`orderers`数组可以包含多个排序节点。如果你有多个排序节点，则应将其全部添加到`orderers`数组中，以便Hyperledger Composer可以与其进行交互。

 7. 我们必须指定一个现有信道的名称。我们会把区块链业务网络部署到信道`composerchannel`中。
```json
"channel": "composerchannel",
```
 8. 最后，我们可以选择指定与区块链业务网络进行交互时，对交易背书的超时时间。
```json
  "timeout": 300
}
```
在这里，我们已经指定了300秒的超时时间。如果一个交易需要超过300秒的时间来背书，那么将会抛出一个超时错误。

 9. 保存你的更改到`connection.json`。完成的连接配置文件应该如下所示：
```json
{
  "name": "fabric-network",
  "type": "hlfv1",
  "mspID": "Org1MSP",
  "peers": [
      {
          "requestURL": "grpc://localhost:7051",
          "eventURL": "grpc://localhost:7053"
      }
  ],
  "ca": {
      "url": "http://localhost:7054",
      "name": "ca.org1.example.com"
  },
  "orderers": [
      {
          "url" : "grpc://localhost:7050"
      }
  ],
  "channel": "composerchannel",
  "timeout": 300
}
```

### 步骤四：定位Hyperledger Fabric管理员的证书和私钥

要将区块链业务网络部署到Hyperledger Fabric网络，我们必须将自己认证为具有执行此操作权限的管理员。在这一步骤中，你可以定位出认证自己为管理员需要的文件。

我们的Hyperledger Fabric网络的管理员是一名叫`Admin@org1.example.com`的用户。该用户的证书和私钥文件存储在以下目录中：
```
~/fabric-tools/fabric-scripts/hlfv11/composer/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
```
你必须首先定位该用户的证书文件。证书是身份的公开部分。证书文件可以在`signcerts`子目录中找到，文件名是`Admin@org1.example.com-cert.pem`。如果你查看这个文件的内容，那么你会发现一个类似于下面的PEM编码证书：
```
-----BEGIN CERTIFICATE-----
MIICGjCCAcCgAwIBAgIRANuOnVN+yd/BGyoX7ioEklQwCgYIKoZIzj0EAwIwczEL
MAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG
cmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHDAaBgNVBAMTE2Nh
Lm9yZzEuZXhhbXBsZS5jb20wHhcNMTcwNjI2MTI0OTI2WhcNMjcwNjI0MTI0OTI2
WjBbMQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMN
U2FuIEZyYW5jaXNjbzEfMB0GA1UEAwwWQWRtaW5Ab3JnMS5leGFtcGxlLmNvbTBZ
MBMGByqGSM49AgEGCCqGSM49AwEHA0IABGu8KxBQ1GkxSTMVoLv7NXiYKWj5t6Dh
WRTJBHnLkWV7lRUfYaKAKFadSii5M7Z7ZpwD8NS7IsMdPR6Z4EyGgwKjTTBLMA4G
A1UdDwEB/wQEAwIHgDAMBgNVHRMBAf8EAjAAMCsGA1UdIwQkMCKAIBmrZau7BIB9
rRLkwKmqpmSecIaOOr0CF6Mi2J5H4aauMAoGCCqGSM49BAMCA0gAMEUCIQC4sKQ6
CEgqbTYe48az95W9/hnZ+7DI5eSnWUwV9vCd/gIgS5K6omNJydoFoEpaEIwM97uS
XVMHPa0iyC497vdNURA=
-----END CERTIFICATE-----
```
接下来，您必须找到该用户的私钥文件。私钥用于以这个身份对交易进行签名。私钥文件可以在`keystore`子目录中找到。私钥文件的名称是一个很长的十六进制字符串，后缀是`_sk`，例如`114aab0e76bf0c78308f89efc4b8c9423e31568da0c340ca187a9b17aa9a4457_sk`。每次生成配置文件名称都会更改。如果你看看这个文件的内容，那么你会发现一个PEM编码的私钥类似于以下内容：
```
-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBG0wawIBAQQg00IwLLBKoi/9ikb6
ZOAV0S1XeNGWllvlFDeczRKQn2uhRANCAARrvCsQUNRpMUkzFaC7+zV4mClo+beg
4VkUyQR5y5Fle5UVH2GigChWnUoouTO2e2acA/DUuyLDHT0emeBMhoMC
-----END PRIVATE KEY-----
```
记住这两个文件的路径，或将它们复制到上一步创建的连接配置文件`connection.json`相同的目录中。你将在下一步骤中需要这些文件。

### 步骤五：为Hyperledger Fabric管理员创建一张业务网络卡片
业务网络卡片包含连接到区块链业务网络和底层Hyperledger Fabric网络所需的全部信息。此信息包括步骤三中创建的连接配置文件，以及步骤四中的管理员的证书和密钥。

在这一步中，你将创建一个业务网络卡片，供管理员用来将区块链业务网络部署到Hyperledger Fabric网络。

运行`composer card create`命令创建一个业务网络卡片。你必须指定在以前的步骤中创建或定位的所有三个文件的路径：
```bash
composer card create -p connection.json -u PeerAdmin -c Admin@org1.example.com-cert.pem -k 114aab0e76bf0c78308f89efc4b8c9423e31568da0c340ca187a9b17aa9a4457_sk -r PeerAdmin -r ChannelAdmin
```
名为`PeerAdmin@fabric-network.card`的业务网络卡片文件将被写入当前目录。我们来探索一下传递给`composer card create`命令的选项。
```
-p connection.json
```
这是我们在第三步创建的连接配置文件的路径。
```
-u PeerAdmin
```
这是我们用来引用管理员用户的名字。代替使用`Admin@org1.example.com`，这个输入起来太长，我们给了一个名字`PeerAdmin`，这样我们就可以很容易地引用这个用户。
```
-c Admin@org1.example.com-cert.pem
```
这是我们在第四步定位的用户`Admin@org1.example.com`的证书文件路径。
```
-k 114aab0e76bf0c78308f89efc4b8c9423e31568da0c340ca187a9b17aa9a4457_sk
```
这是我们在第四步找到的用户`Admin@org1.example.com`的私钥文件路径。
```
-r PeerAdmin -r ChannelAdmin
```
在这里，我们指定用户拥有哪些角色。此信息是必需的，以便Hyperledger Composer知道哪些用户能够执行哪些操作。用户`Admin@org1.example.com`是Hyperledger Fabric网络的管理员，具有角色`PeerAdmin`（能够安装链码）和角色`ChannelAdmin`（能够实例化链码）。

### 步骤六：为Hyperledger Fabric管理员导入业务网络卡片
Hyperledger Composer只能使用放置在钱包中的业务网络卡片。钱包是包含业务网络卡片的文件系统上的目录。在这一步中，你将将步骤五中创建的业务网络卡片导入到钱包中，以便你可以在后续步骤中使用这个业务网络卡片。

运行`composer card import`命令将业务网络卡片导入钱包：
```bash
composer card import -f PeerAdmin@fabric-network.card
```
我们来探索一下传递给`composer card import`命令的选项。
```
-f PeerAdmin@fabric-network.card
```
这是我们在第五步创建的业务网络卡片文件的路径。

你现在可以通过引用名称`PeerAdmin@fabric-network`来使用此业务网络卡片。你现在已经准备好将区块链业务网络部署到Hyperledger Fabric网络。

我们将部署区块链业务网络`tutorial-network`，这是按照[开发者教程](tutorials_developer-tutorial.md)创建的。

### 步骤七：将Hyperledger Composer运行时安装到Hyperledger Fabric peer节点上
Hyperledger Composer包含一个名为“Hyperledger Composer运行时”的组件，该组件提供用于托管和支持业务网络存档的所有功能，例如数据验证、错误处理、交易处理函数执行和访问控制。在Hyperledger Fabric术语中，Hyperledger Composer运行时是一个标准的链码。

在这一步中，你将在所有Hyperledger Fabric peer节点上安装Hyperledger Composer运行时。在Hyperledger Fabric中，这是一个链码安装操作。

运行以下`composer runtime install`命令将Hyperledger Composer运行时安装到你在步骤三中创建的连接配置文件中指定的所有Hyperledger Fabric peer节点上：
```bash
composer runtime install -c PeerAdmin@fabric-network -n tutorial-network
```
我们来探索一下传递给`composer runtime install`命令的选项。
```
-c PeerAdmin@fabric-network
```
这是我们在步骤六中导入到钱包中的业务网络卡片的名称。
```
-n tutorial-network
```
你必须为每个区块链业务网络安装Hyperledger Composer运行时的副本，并指定区块链业务网络的名称。在这里，我们指定正在部署的区块链业务网络的名称`tutorial-network`。

### 步骤八：启动区块链业务网络
在这一步中，你将启动区块链业务网络。在Hyperledger Fabric术语中，这是链码实例化操作。

运行`composer network start`命令启动区块链业务网络：
```bash
composer network start -c PeerAdmin@fabric-network -a tutorial-network.bna -A admin -S adminpw
```
我们来探索一下传递给`composer network start`命令的选项。
```
-c PeerAdmin@fabric-network
```
这是我们在步骤六中导入到钱包中的业务网络卡片的名称。
```
-a tutorial-network.bna
```
这是指向业务网络存档的路径，其中包含了我们名为`tutorial-network`的区块链业务网络的业务网络定义。
```
-A admin
```
在部署区块链业务网络时，你必须创建至少一个将成为区块链业务网络管理员的参与者。此参与者负责将其他参与者加入区块链业务网络。在这里，我们正在指定要创建一个叫做`admin`的区块链业务网络管理员。
```
-S adminpw
```
这指定我们的区块链业务网络管理员`admin`将使用登记密码`adminpw`从CA（证书颁发机构）请求证书和私钥。指定此选项时，为业务网络管理员指定的用户名称必须是一个已登记的ID(已经向CA注册)。

现在我们的区块链业务网络已经启动，我们可以使用已创建的业务网络卡片文件`admin@tutorial-network.card`与它进行交互。

### 步骤九：为业务网络管理员导入业务网络卡片
运行`composer card import`命令将业务网络卡片导入钱包：
```bash
composer card import -f admin@tutorial-network.card
```
你现在可以通过指定名称`admin@tutorial-network`来使用此业务网络卡片。现在你已经准备好与正在运行的区块链业务网络进行交互了！

### 步骤十：测试与区块链业务网络的连接
运行`composer network ping`命令测试与区块链业务网络的连接：
```bash
composer network ping -c admin@tutorial-network
```
### 结论
在本教程中，你已经了解了如何配置Hyperledger Composer以及连接到Hyperledger Fabric网络所需的所有信息，以及如何将区块链业务网络部署到Hyperledger Fabric网络。

如果你使用了开发环境中提供的简单Hyperledger Fabric网络，那么为什么不尝试按照Hyperledger Fabric文档构建自己的Hyperledger Fabric网络，并查看是否可以成功部署区块链业务网络？
