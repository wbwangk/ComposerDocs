# 使用Hyperledger Composer进行安装和开发

按照以下说明获取所需的Hyperledger Composer开发工具并建立Hyperledger Fabric。

## 在你开始之前

要运行Hyperledger Composer和Hyperledger Fabric，我们建议你至少拥有4Gb的内存。

以下是安装所需开发工具的先决条件：

- 操作系统：Ubuntu Linux 14.04 / 16.04 LTS（均为64位）或Mac OS 10.12

- Docker引擎：版本17.03或更高

- Docker-Compose：版本1.8或更高

- Node：8.9或更高（注意版本9不支持）

- npm：v5.x

- git：2.9.x或更高版本

- Python：2.7.x

- 你选择的代码编辑器，我们推荐VSCode。

**如果使用Linux安装Hyperledger Composer，请注意以下建议：**

- 以普通用户身份登录，而不是root用户。

- 不要`su`到root。

- 安装先决条件时，使用curl，然后使用sudo进行解压缩。

- 以普通用户身份运行prereqs-ubuntu.sh。它可能会提示输入root密码，因为它的某些操作需要以root身份运行。

- 不要 `sudo`使用npm或`su`root来使用它。

- 避免以root身份全局安装node。

如果你在Ubuntu上运行，则可以使用以下命令下载先决条件：
```bash
curl -O https://hyperledger.github.io/composer/prereqs-ubuntu.sh
chmod u+x prereqs-ubuntu.sh
```

接下来运行这个脚本 - 因为这个脚本在执行过程中使用了sudo，所以会提示你输入密码。
```bash
./prereqs-ubuntu.sh
```

如果你正在运行Mac OS X，则可以按照[Mac OS X先决条件](https://hyperledger.github.io/composer/installing/prereqs-mac.html)的[安装指南进行操作](https://hyperledger.github.io/composer/installing/prereqs-mac.html)。

## 步骤1：安装Hyperledger Composer开发工具

你将需要的开发工具都可以用`npm install -g`安装（作为非特权用户，例如非root用户）。

1.要安装`composer-cli`运行以下命令：
```bash
npm install -g composer-cli
```

`composer-cli`包含了用于开发业务网络的所有命令行操作。

2.要安装`generator-hyperledger-composer`运行以下命令：
```bash
npm install -g generator-hyperledger-composer
```

这`generator-hyperledger-composer`是一个Yeoman插件，为你的业务网络创建定制的应用程序。

3.要安装`composer-rest-server`运行以下命令：
```bash
npm install -g composer-rest-server
```

   在`composer-rest-server`使用Hyperledger Composer LoopBack连接器连接到一个业务网络，提取模型，然后呈现一个页面，页面包含了按模型生成的REST API。

4.要安装`Yeoman`运行以下命令：
```bash
npm install -g yo
```

   Yeoman是一个生成应用程序的工具。与`generator-hyperledger-composer`组件结合使用时，它可以解释业务网络并基于它们生成应用程序。

### 可选的开发工具

1.如果你使用VSCode，请从VSCode市场安装Hyperledger Composer VSCode插件。

2.如果要在本地使用Playground运行连接到业务网络，使用以下命令安装`composer-playground`。

```bash
npm install -g composer-playground
```

3.要在本地运行Playground，请运行以下命令：

```bash
composer-playground
```

   Playground会在以下地址自动打开：[http://localhost:8080/login](http://localhost:8080/login)

## 步骤2：启动Hyperledger Fabric

如果你在[本地安装了Hyperledger Composer Playground](installing_using-playground-locally.md)，则需要使用以下脚本关闭容器。

> *请注意：这些命令将终止并删除所有正在运行的容器，并应删除所有以前创建的Hyperledger Fabric链码镜像。*
```bash
docker kill $(docker ps -q)
docker rm $(docker ps -aq)
docker rmi $(docker images dev-* -q)
```

1.在你选择的目录中（假设`~/fabric-tools`）获得包含安装Hyperledger Fabric v1.0的工具的zip文件。
```bash
mkdir ~/fabric-tools && cd ~/fabric-tools
curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.zip
unzip fabric-dev-servers.zip
```

   还可以选择`tar.gz`文件
```bash
mkdir ~/fabric-tools && cd ~/fabric-tools
curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.tar.gz
tar xvzf fabric-dev-servers.tar.gz
```

2.如果这是第一次，则需要先下载fabric运行时。如果你已经下载了它，请启动fabric环境，并创建一个Hyperledger Composer profile（配置文件）。之后，你可以选择停止该fabric，然后再次启动。你也可以彻底清理Hyperledger Fabric和Hyperledger Composer配置文件。

   所有脚本将在目录`~/fabric-tools` 中。使用Hyperledger Composer的典型顺序是
```bash
cd ~/fabric-tools
./downloadFabric.sh
./startFabric.sh
./createPeerAdminCard.sh
   ```
   然后在开发阶段结束
```bash
cd ~/fabric-tools
./stopFabric.sh
./teardownFabric.sh
```

> 请注意：所创建的开发环境将包含一个`PeerAdmin`身份，包括部署业务网络所需的加密材料。

## 脚本细节

![img](https://hyperledger.github.io/composer/assets/img/developer-tools-commands.png)。

此图解释了脚本可能运行的顺序。

**下载Fabric**

从`fabric-tools`目录运行`./downloadFabric.sh`

**开始Fabric**

从`fabric-tools`目录运行`./startFabric.sh`

**停止Fabric**

从`fabric-tools`目录运行`./stopFabric.sh`

**创建PeerAdmin卡**

为正在运行的Hyperledger Fabric实例的peer管理员创建一个业务网络卡片，运行 `./createPeerAdminCard.sh`

注意：这将创建并导入一个业务网络卡片以连接到你已经启动的开发fabric。

**卸载Fabric**

从`fabric-tools`目录运行`./teardownFabric.sh`

## 接下来是什么？

- 开始[**写一个业务网络的定义**](business-network_business-network-index.md)。
- 如果你正在查找使用开发人员工具的教程，请参阅[**开发人员指南**](tutorials_developer-tutorial.md)以使用开发人员工具运行一个示例。
