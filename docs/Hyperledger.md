## Hyperledger概述
### 什么是区块链
#### 一个分布式账本
所有事务数据被保存到账本中。账本中的记录只能增加，不能修改。所有节点(Peer)都保存了事务数据的完整副本。每间隔一定时间或交易量达到一定数量，形成一个区块(Block)。区块一旦形成就不能修改。多个区块顺序链接在一起形成区块链。  
（把一段时间内生成的信息（包括数据或代码）打包成一个区块，盖上时间戳，与上一个区块衔接在一起，每下一个区块的页首都包含了上一个区块的索引数据，然后再在本页中写入新的信息，从而形成新的区块，首尾相连，最终形成了区块链。）  
事务日志保存到区块链中（有点像数据库日志），账本的当前状态体现在世界状态中。世界状态保存在NoSQL数据库中，如Leveldb、Couchdb。  
#### 智能合约(smart contract)
在Hyperldedger Fabric中链码(chaincode)和智能合约(smart contract)是一个意思。  
链码是封装用于创建和修改资产的业务逻辑和事务处理指令的软件。当外部应用程序需要与账本交互时，它需要调用链码。在大多数情况下，链码仅与账本的的数据库组件-世界状态（例如查询）交互，而不是与事务日志(区块链)交互。  
链码可以用几种编程语言来实现。当前支持的链式代码语言是Go和node.js，支持未来发行版中的Java和其他语言。  
Hyperledger Fabric支持私有网络（使用通道）。每个节点（参与者或称成员）可以加入多个通道，通道内的节点同享同一个账本。  
具有正确权限的用户可以轻松地安装和实例化通道的链式代码，并查看是谁在他们参与的渠道中，恰当授权的用户可以根据所建立的区块链网络的策略调用链码，创建新的通道，甚至更新通道的访问权限。  
#### 共识
在整个网络中保持账本交易同步的过程 - 确保账本只有在交易得到适当的参与者批准时才会更新，而且当账本更新时，它们以相同的顺序更新相同的交易 - 被称为共识。  
将共识定义为三阶段流程：认可，排序和生效证。
现在，将区块链视为一个共享的，复制的事务系统就足够了，该系统通过智能合约进行更新，并通过一个称为共识的协作过程保持一致的同步。  
事务必须按照发生的顺序写入账本。共识算法有很多。例如，PBFT（Practical Byzantine Fault Tolerance）可以提供文件副本相互通信的机制，以保持每个副本的一致性，即使在发生损坏的情况下也是如此。  
Hyperledger Fabric被设计为允许网络发起者（指通道建立者？）选择最能代表参与者之间存在关系的共识机制。与隐私一样，还有一系列需求。从具有高度结构化的关系网络到更加对等的网络。  
我们将更多地了解Hyperledger Fabric共识机制，目前包括SOLO和Kafka，并且很快将扩展到SBFT（简化的拜占庭容错）。  

### Hyperledger Projects
5个主项目：Cello、Fabric(Golang)、SawtoothLake(Python)、Iroha、Blockchain Explorer  
- Fabric  
    区块链技术的⼀个实现(Golang)  
- STL-SawtoothLake  
    ⾼度模块化的分布式账本平台（Python）  
    PoET consensus、Transaction Families、Scalability    
- Iroha  
    轻量级的分布式账本， 侧重于移动(C++)
- Blockchain Explorer
    展⽰和查询区块链块、事务和相关数据的Web应⽤(node.js)    
    UI to interact with ledger，Under-development  
- Cello
    BaaS的⼯具集，帮助创建、管理、终⽌区块链(Blockchain as a Service)  

### Fabric
Hyperledger Fabric网络的成员通过成员资格服务提供商（MSP）注册，而不是一个允许未知身份参与网络的开放式无权限系统（需要证明工作证明的协议来验证交易和保护网络）。
Hyperledger Fabric还提供了创建**通道**的能力，允许一组参与者创建单独的交易账本。用通道来控制访问账本的权限。账本可以加入通道，参与者也可以加入通道。对于一个通道，里面的参与者可以“看到”里面的账本。  
#### 成员管理(Membership)
会员注册、⾝份保护、 内容保密、交易审计功能，以保证平台访问的安全性。  
在Hyperledger Fabric中，每个参与者（客户端，peer，oderer）都属于某个组织。  
组织拥有CA，为其成员（客户端，peer，oderer）提供证书，以便向其他人或其他组织认证自己。  
交易方持有多种类型的证书，交易不同环节将使用如下这些类型的证书：  
- 注册证书(Enrollment Cert)。用于验证使用者身份，永不过期
- 事务证书(Transaction Cert)。每个交易时生成，用于交易的签名  
- 通信证书(TLS-Cert)，用于通信通道加密，永不过期  

#### 区块服务（BlockChain&Transaction)
负责节点间的共识管理、账本的分布式计算、账本的存储以及节点间的P2P协议功能的实现，是区块链的核⼼组成部分，为区块链的主体功能提供了底层⽀撑。  
#### 链码(ChainCode)
chaincode即智能合约(smart contract)。该模块为链码提供安全的部署、运⾏的环境。  
#### Event
贯穿于其他各个组件中间，为各个组件间的异步通信提供了技术实现。  
### 账本
由两个组件组成：世界状态和事务日志。  
账本提供了一个所有状态变化的历史记录。它可以是一个业务的系统记录。每个Hyperledger Fabric网络的参与者都拥有账本的副本（网络指通道）。  
**区块账本(Block ledger)**  
 - 基于文件系统，只能添加新区块  
 - 区块保存在所有提交者节点，部分可选子集保存在排序服务节点  
**状态账本**  
 - 世界状态(账本)保存了智能合约数据的当前值  
   如vehicleOwner=Daisy  
 - 开发者通过链码API访问隐藏起来的KVS(key-value数据库)  
   如GetState(), PutState(), GetStateByRange()等等  
 - 保存在所有提交者节点  
**历史账本**  
 - 保存所有链码事务的顺序历史记录。索引保存在KVS，开发者通过链码API访问  
 - 保存在所有提交者节点  
 
## 开发环境(vagrant)
(/e/vagrant10/ambari-agrant/fabric/devenv)   
fabric官方库提供了一个Vagrantfile，是个ubuntu16的环境，供开发调试用。如何想在一个空linux下手工安装，可以参考[Fabri Getting Started](http://hyperledger-fabric.readthedocs.io/en/latest/getting_started.html)。   
  
在windows下启动git bash，然后克隆fabric库：
```
$ cd /e/vagrant10/ambari-vagrant
$ git clone https://github.com/hyperledger/fabric.git
$ cd fabric/devenv
$ vagrant up
$ vagrant ssh
```
虚拟机启动过程中会自动执行一个`setup.sh`脚本进行初始化。有时会半途执行失败。可以进入linux后手工执行脚本：
```
$ cd /hyperledger/devenv
$ ./setup.sh
```
这样，你就拥有了一个fabric开发环境。(snap a2)  

通过分析`setup.sh`，也可以看到手工安装docker-compose、golang的办法：
#### 安装docker-compose
```
$ curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```
#### 安装go lang
`setup.sh`中提供的是从`golang.org`下载安装包，下面描述的是从另一个网站下载安装包。
```
$ cd /opt
$ wget https://redirector.gvt1.com/edgedl/go/go1.9.2.linux-amd64.tar.gz
$ tar -C /usr/local -xzf go1.9.2.linux-amd64.tar.gz
```
在文件`/etc/profile`的最后添加：
```
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/opt/gopath
```
使环境变量生效后验证go版本：
```
$ source /etc/profile
$ go version
go version go1.9.2 linux/amd64
```
### Hyperledger Fabric Samples
[Hyperledger官方GETTING STARTED](https://hyperledger-fabric.readthedocs.io/en/latest/samples.html)  
```
$ cd /opt
$ git clone -b master https://github.com/hyperledger/fabric-samples.git
$ cd fabric-samples
$ curl -sSL https://goo.gl/6wtTN5 | bash -s 1.1.0-preview
```
脚本会自动下载了很多docker镜像:
```
hyperledger/fabric-ca          x86_64-1.1.0-preview   2736904862db        4 weeks ago         218MB
hyperledger/fabric-tools       x86_64-1.0.4           6051774928a6        4 weeks ago         1.33GB
hyperledger/fabric-couchdb     x86_64-1.0.4           cf24b91dfeb1        4 weeks ago         1.5GB
hyperledger/fabric-kafka       x86_64-1.0.4           7a9d6f3c4a7c        4 weeks ago         1.29GB
hyperledger/fabric-zookeeper   x86_64-1.0.4           53c4a0d95fd4        4 weeks ago         1.3GB
hyperledger/fabric-orderer     x86_64-1.0.4           b17741e7b036        4 weeks ago         151MB
hyperledger/fabric-peer        x86_64-1.0.4           1ce935adc397        4 weeks ago         154MB
hyperledger/fabric-javaenv     x86_64-1.0.4           a517b70135c7        4 weeks ago         1.41GB
hyperledger/fabric-ccenv       x86_64-1.0.4           856061b1fed7        4 weeks ago         1.28GB
```
上述脚本还下载了几个工具到`/opt/fabric-samples/bin`目录下，它们是cryptogen,configtxgen,configtxlator,peer。  
为了访问它们方便，需要修改PATH参数。编辑`/etc/profile`，在文件的最后添加内容：
```
export PATH=/opt/fabric-samples/bin:$PATH
```
为了保存VM当前状态，回到vagrant命令行下执行`vagrant snapshot save a3`  

## 启动首个网络(first-network)
本节遵循hyperledger官方文档“[Building Your First Network](http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html)  
first-network提供了一个脚本来帮助初学者体验fabric，它就是`byfn.sh`。可以通过帮助命令看看它的功能：
```
$ cd /opt/fabric-samples/first-network
$ ./byfn.sh --help
 byfn.sh -m up|down|restart|generate [-c <channel name>] [-t <timeout>] [-d <delay>] [-f <docker-compose-file>] [-s <dbtype>]
```
### 生成网络工件
generate required certificates and genesis block
fabric的网络和通道具有类似的含义。通道可以视为一种虚拟网络，多个通道就多个虚拟网络。docker支持overlay网络，为同一个peer加入不同的网络创造了底层技术基础（以上认识还没有得到确认）。
```
$ ./byfn.sh -m generate
Generating certs and genesis block for with channel 'mychannel' and CLI timeout of '10000'
Continue (y/n)?y
(后略)
```
`byfn.sh`会在屏幕上有很多输出，通过这些文字也可以知道该脚本的功能：
1. 生成了Orderer的创世区块。在排序节点上fabric维护了一个“系统账本”，保存了整个fabric区块链网络的参数、元数据等。对于区块链来说，第一个块(有时称0号区块)被称为创世区块(Genesis block)。该创世区块对应了一个文件，一般是`genesis.block`。  
2. 创建了一个通道。生成了一个叫`channel.tx`的文件。  
3. 生成了组织Org1MSP和Org2MSP的锚peer。锚peer用于跨组织的通信。  

### 启动网络
```
$ ./byfn.sh -m up
（适当删减）
Channel "mychannel" is created successfully =====================
PEER0 joined on the channel "mychannel" =====================
PEER1 joined on the channel "mychannel" =====================
PEER2 joined on the channel "mychannel" =====================
PEER3 joined on the channel "mychannel" =====================
Anchor peers for org "Org1MSP" on "mychannel" is updated successfully =====================
Anchor peers for org "Org2MSP" on "mychannel" is updated successfully =====================
Chaincode is installed on remote peer PEER0 =====================
Chaincode is installed on remote peer PEER2 =====================
Chaincode Instantiation on PEER2 on channel 'mychannel' is successful =====================
Querying on PEER0 on channel 'mychannel'... =====================
Invoke transaction on PEER0 on channel 'mychannel' is successful =====================
Chaincode is installed on remote peer PEER3 =====================
Querying on PEER3 on channel 'mychannel'... =====================
========= All GOOD, BYFN execution completed ===========
```
启动后屏幕仍被锁定为日志输出，可以打开另外的终端窗口进行后续的操作。如，查看容器清单：
```
$ docker ps
```
可以看到容器有：
1. cli(fabric-tools)，是fabric的命令行工具  
2. peer0.org1.example.com等(fabric-peer)，是peer节点的进程容器  
3. orderer.example.com(fabric-orderer)，是orderer节点的进程容器  
4. dev-peer0.org1.example.com-mycc-1.0-xxxx， 是链码容器
在其他的fabric环境中(如生产环境下)，还可能看到ca-server的容器、couchdb的容器等。  

### 停止网络
```
$ ./byfn.sh -m down
$ docker ps
```
网络停止后，通过docker ps命令可以看到所有的容器都消失了。

### 密钥生成器(Crypto Generator)
我们用`cryptogen`工具为各种的网络实体生成密码学文件(x509证书和签名密钥)。这些证书表达身份，对实体间通信和交易认证进行签名和验证。  
Cryptogen的配置文件是`crypto-config.yaml`，该文件包括网络拓扑，允许我们为组织以及属于组织的组件生成一系列证书和密钥。执行示范：
```
$ cryptogen generate --config=./crypto-config.yaml
```
运行后会自动创建一个`crypto-config`目录，里面有很多密码学文件。在生成的文件中，每个组织都会分配一个根证书(`ca-cert`)，该证书绑定特殊组件(peer和orderer)到组织。假定每个组织都有一个唯一的CA证书，我们模仿了一个典型的网络，其中每个[成员](http://hyperledger-fabric.readthedocs.io/en/latest/glossary.html#member)拥有自己的CA。在Hyperledger Fabric中，实体使用自己的私钥(`keystore`)对事务和通信进行签名，并用对方的公钥(`signcerts`)验证签名。  
配置文件中有个`count`变量，我们用它来指定每个组织下的peer数量；在我们示例中，每个组织下有两个peer。我们在本文不会详述[X509证书和PKI](https://en.wikipedia.org/wiki/Public_key_infrastructure)。  
在`crypto-config.yaml`文件中，注意`OrdererOrgs`之下的“Name”, “Domain” and “Specs”参数。网络实体的命名约定是：`{{.Hostname}}.{{.Domain}}`。例如，排序节点的名称是`orderer.example.com`，这关联了一个MSP ID `Orderer`，关于MSP的更多细节参考[Membership Service Providers (MSP)](http://hyperledger-fabric.readthedocs.io/en/latest/msp.html)文档。  
 
### 配置交易生成器
工具`configtxgen`用于生成四个配置工件：
- 排序器(orderer)`genesis block`
- 通道`configuration transaction`  
- 两个`anchor peer transactions`，每个组织生成一个  
关于`configtxgen`更多细节参考[Channel Configuration (configtxgen)](http://hyperledger-fabric.readthedocs.io/en/latest/configtxgen.html)。  

生成的四个工件中的`genesis block`是排序服务的[创世区块](#创世区块)。[通道](#通道)的`configuration transaction`文件在通道创建时被广播到orderer。至于`anchor peer transactions`，顾名思义，定义组织在这个通道的[锚点peer](#锚点peer)。  

#### 工作原理

Configtxgen会根据文件`configtx.yaml`定义的配置运行，文件包含了示范网络的定义。其中有三个成员：一个排序器组织(`OrdererOrg`)和两个Peer组织(`Org1`和`Org2`)，每个Peer组织管理和维护了两个peer节点。这个文件还定义了一个联盟(`SampleConsortium`)，联盟包含了两个peer组织。特别需要注意文件开头的“Profiles”小节。你应该注意到了它有两个唯一的头。一个是排序器创世区块(`TwoOrgsOrdererGenesis`)，另一个是通道(`TwoOrgsChannel`)。  

这些头很重要，当生成工件时，它们将作为参数发送。  
*注意：我们的`SampleConsortium`联盟定义在系统级profile，然后被通道集profile引用。通道将存在于整个联盟范围。*  

这个文件还包含了两个值得注意的附加内容。首先，为每个peer组织(`peer0.org1.example.com`和`peer0.org2.example.com`)定义了锚点peer。其次，定义了每个成员的MSP目录地址，这让我们在排序器创世区块中保存了每个组织的根证书。这是一个关键概念。现在与排序服务通信的任何网络实体可以被验证其数字签名了。  

### 运行工具
你可以使用`configtxgen`和`cryptogen`工具手工生成证书/密钥和不同的配置工件。或者，你也可以修改`byfn.sh`脚本来达到你的目的。  

#### 手工生成工件
你可以参考byfn.sh脚本中的generateCerts函数，里面的命令可以根据`crypto-config.yaml`文件中定义的网络配置生成证书。  

首先，让我们运行`cryptogen`工具。它位于`bin`目录，所以要使用相对路径了执行它（或者修改操作系统的PATH）:
```
$ ../bin/cryptogen generate --config=./crypto-config.yaml
org1.example.com
org2.example.com
```
证书和密钥(即MSP文书)会被输出到`crypto-config`目录（位于`first-network`目录之下）。  

下一步，我们需要告诉`configtxgen`工具到哪里去寻找`configtx.yaml`文件。需要通过下面的环境变量告诉它($PWD表示当前目录)：
```
$ export FABRIC_CFG_PATH=$PWD
```
然后，我们调用`configtxgen`工具去生成排序器创世区块：
```
$ ../bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
```
需要先手工创建目录`channel-artifacts`，否则上述命令会出错。  
#### 创建一个通道配置事务
（在Fabric中通道配置信息也保存在区块链中，而区块链的内容是靠事务写入的，所以要新建通道需要创建一个事务。）  
下一步，我们需要创建通道事务工件。确保替换`$CHANNEL_NAME`或设置`CHANNEL_NAME`为环境变量，然后执行下列指令：
```
$ export CHANNEL_NAME=mychannel  
$ ../bin/configtxgen -profile TwoOrgsChannel \
 -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
```
下一步，我们将在刚刚创建的通道上定义Org1的锚点peer。同样，确保覆盖`$CHANNEL_NAME`或设置为下面的命令设置环境变量。
```
$ ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate \
./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
2017-12-07 08:59:42.756 UTC [common/tools/configtxgen] main -> INFO 001 Loading configuration
2017-12-07 08:59:42.762 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
2017-12-07 08:59:42.762 UTC [common/tools/configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update
```
现在，我们将在同一个通道中定义Org2的锚点peer：
```
$ ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate \
./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP
```
总结一下本节，执行了上述一系列命令后，`crypto-config`目录生成了一些证书和密钥；`channel-artifacts`下生成了一个创世区块文件和3个事务文件：
```
$ ls ./channel-artifacts
channel.tx  genesis.block  Org1MSPanchors.tx  Org2MSPanchors.tx
```
### 启动网络
我们将使用docker-compose脚本启动我们的网络。docker-compose会使用我们之前下载的docker镜像，用我们之前生成`genesis.block`(创世区块)引导排序器(orderer)。  
在启动网络前，打开`docker-compose-cli.yaml`文件，注释掉CLI容器的`script.sh`。让你的`docker-compose-cli.yaml`文件变成这样：
```
working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
# command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME}; sleep $TIMEOUT'
volumes
```
如果不注释掉这一样，脚本就会利用CLI命令把网络启动起来了。然而，我们的目的是手工执行这些命令，以便解释这些调用的语法和功能。  
CLI默认超时是10000秒。如果你需要容器存在的更久，需要通过设置`TIMEOUT`环境变量覆盖这一默认值。  

启动网络(确保用命令`docker ps -a`看不到任何容器)：
```
$ TIMEOUT=10000 CHANNEL_NAME=$CHANNEL_NAME docker-compose -f docker-compose-cli.yaml up -d
```
如果你想看到网络的实时日志，就不要加上`-d`标志。如果不加`-d`表示，你需要另外打开一个终端窗口了执行CLI。  

#### 环境变量
为了通过CLI命令让`peer0.org1.example.com`工作起来，我们需要准备4个环境变量。这些变量之前被“烧入”了`peer0.org1.example.com`CLI容器中，所以我们不用输入它们就能操作。但如果你需要调用其他peer或orderer，则需要正确的设置这些变量值。查看`docker-compose-base.yaml`文件可以看到这四个变量的值：
```
# Environment variables for PEER0

CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```
#### 创建和加入通道
回想一下上面我们在[创建一个通道配置事务](#创建一个通道配置事务)一节中使用`configtxgen`工具创建通道配置事务。你可以重复那个过程来创建另外的通道配置事务，使用相同或不同的profile参数(在`configtx.yaml`中定义)传递给`configtxgen`。你可以重复这一过程，用来在网络中创建其他通道。  
用下列命令进入CLI容器：
```
$ docker exec -it cli bash
root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer#
```
首先进入的是在`docker-compose.ymal`中定义的`working_dir`目录。下面用`$$`表示在容器内的命令行操作。  
在之前[创建一个通道配置事务](#创建一个通道配置事务)一节中我们创建了通道配置事务工件("channel.txt")，下面我们把它作为创建通道请求的一部分发给orderer。  
我们用`-c`标志指定通道名称，用`-f`标志指定通道配置事务。在这里它叫`channel.txt`，然而你可以用其他名字来挂载你自己的通道配置事务。我们又一次在CLI容器内设置`CHANNEL_NAME`环境变量，所以不用显式地传递这个参数。  
```
$$ export CHANNEL_NAME=mychannel
$$ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
$$ peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f \
./channel-artifacts/channel.tx --tls --cafile $ORDERER_CA
```
注意此命令行中的`--cafile`参数，它是一个指向orderer根CA证书的本地路径，用于验证TLS握手。  
命令会返回一个创世区块(`<channel-ID.block>`)，我们用它加入通道。它里面包含了在`channel.tx`中定义的配置信息，如果你没有改变过通道名称，该命令将返回一个叫`mychannel.block`的原型。在当前目录下可以看到这个`mychannel.block`文件。  
现在，让我们把`peer0.org1.example.com`节点加入通道：
```
$$ peer channel join -b mychannel.block
```
你还可以将其他peer加入通道，方法是修改之前在[环境变量](#环境变量)一节中提到的4个环境变量。如果你用`env`命令查看一下环境变量，会发现那4个环境变量的值都是`peer0.org1.example.com`对应的。  
我们不将每个peer都加入网络，而是将`peer0.org2.example.com`加入网络，这样我们可以修改通道的锚点peer定义。我们用下面的命令将预先烧制在CLI容器中的4个环境变量替换掉:
```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer channel join -b mychannel.block
```
#### 更新锚点peer 
下面的命令是通道变更，他们会广播到通道定义。本质上，我们会在通道创始区块的上面追加配置信息。注意，我们没有改变创始区块，只是向链中添加了定义锚点peer的delta。  
变更通道定义，将`peer0.org1.example.com`定义为Org1的锚点peer。
```
peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```
现在，变更通道定义，将`peer0.org2.example.com`定义为Org2的锚点peer。注意命令中的4个环境变量用于覆盖默认的`peer0.org1.example.com`的相关值。
```
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```
#### 安装和实例化链码
*注意，我们简单实用了一个已有链码。如果想学习链码开发，请参考[链码开发](链码教程:链码开发)一章。*  
应用通过链码与区块链账本交互。我们需要安装链码到那些执行和为事务背书的peer上，并在通道上实例化链码。  
首先，安装Go语言示范链码到4个peer节点的某个上。这些命令会将源码放到指定peer的文件系统上。  
*注意，对于每个链码名称和版本，你只能安装一个版本的源码。源码以链码的名称和版本号为上下文存在于peer的文件系统中，它不关注语言。*  
```
$$ peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
```
下一步在通道上实例化链码。这将在通道上初始化链码、为链码设置背书策略和为目标peer启动一个链码容器。注意`-P`参数。这是链码的背书策略，用于对链码事务进行验证。  
在下面的命令中，你注意到了我们将策略设置为`-P "OR ('Org0MSP.member','Org1MSP.member')"`。这意味着我们需要从Org1或Org2的peer上获得一个背书。如果把`OR`改成`AND`，则表示我们需要两个背书。  
```
$$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
```
关于背书策略的细节可以看到[背书策略](http://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html)。  
如果你想更多的peer与账本交互，你需要将它们加入通道，安装同样名字、版本和语言的链码到相应peer的文件系统。一个链码容器被在peer上启动，然后就可以与相应链码交互了。需要知道的是，Node.js镜像的编译相对较慢。  
当链码在通道上被实例化后，我们只需传入通道id和链码名称来访问它。  
#### 查询
让我们查询一下键`a`的值，以便确认链码已经实例化和状态数据库已经填充。查询语句如下：
```
$$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
Query Result: 100    (其它信息省略)
```
#### 调用(Inoke)
现在让我们把`a`的10个给`b`（即a减少10，b增加10）。这个事务会切割一个新区块并更新状态数据库。调用的语法如下：
```
$ peer chaincode invoke -o orderer.example.com:7050  --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'
``` 
然后分别查询一下`a`和`b`的值：
```
$$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
Query Result: 90    (其它信息省略)
$$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","b"]}'
Query Result: 210    (其它信息省略)
```
#### 这演示了什么？
为了对账本进行读写操作，peer必须安装链码。此外，链码容器并没有启动，直到对链码进行初始化或执行读写事务(如查询`a`的值)。这些事务促使容器启动。而且，通道中的所有peer会维持一个账本的完全副本，其中包括不可修改、区块中的顺序记录，以及状态数据库(其中维护了当前状态的快照)。这包含没有安装链码的peer（就像上面例子中的`peer1.org1.example.com`peer)。 最终，链码在安装后可以访问(就像上面例子中的`peer1.org2.example.com`)，因为它已经被实例化。  
#### 怎么看到这些事务？
检查CLI docker容器的日志(需要先通过exit命令先退出容器，返回到宿主操作系统):
```
$ docker logs -f cli
```
但，我的cli容器的日志是空的！原因不明
#### 怎么看到链码日志？
用`docker logs`命令查看不同链码容器的日志，来分别查看各个事务的日志。需要先用`docker ps`命令找到链码容器id。下面是刚刚测试的链码容器日志：
```
docker logs 3daea3abfab2
ex02 Init
Aval = 100, Bval = 200
ex02 Invoke
Query Response:{"Name":"a","Amount":"100"}
ex02 Invoke
Aval = 90, Bval = 210
ex02 Invoke
Query Response:{"Name":"a","Amount":"90"}
ex02 Invoke
Query Response:{"Name":"b","Amount":"210"}
```
### 理解docker-comopse拓扑
BYFN范例提供了两种风格的Docker Compose文件，都是从`docker-compose-base.yaml`(位于`base`目录)扩展而来。第一种风格是`docker-compose-cli.yaml`，提供了一个CLI容器,以及一个orderer和4个peer。我们在这个文章中主要使用这个文件。  
注意：本文剩余部分的内容主要讲一个为了SDK设计的docker-compose文件。更多细节参考[Node SDK库](https://github.com/hyperledger/fabric-sdk-node)。*  
第二种风格的是`docker-compose-e2e.yaml`，这个用于使用Node.js SDK进行的端到端测试。为了使用SDK，它的主要不同是包含一个运行fabric-ca服务器的容器。因此，我们可以发送REST请求到组织的CA，用来进行用户的登记(registration)和注册(enrollment)。  
如果你想使用`docker-compose-e2e.yaml`而不运行`byfn.sh`脚本，需要进行4个小修改。我们需要指出组织CA的私钥。你可以指出私钥在crypto-config目录中的位置。例如，Org1的私钥是路径`crypto-config/peerOrganizations/org1.example.com/ca/`。这个私钥的文件名是一个以`_sk`结尾的长哈希值。Org2的私钥路径是`crypto-config/peerOrganizations/org2.example.com/ca/`。  
在`docker-compose-e2e.yaml`中为ca0和ca1修改`FABRIC_CA_SERVER_TLS_KEYFILE`变量。你还需要修改启动ca服务器的命令路径。你需要为每个CA容器提供同样的私钥两次。  
### 使用CouchDB
状态数据库可以从默认(goleveldb)切换到CouchDB。同样的链码函数可以用于CouchDB，然而，当把链码数据建模为JSON后，还可以对状态数据库的数据内容执行丰富而复杂的查询。  
使用CouchDB代替默认数据库(goleveldb)，与之前描述相同步骤生成工件，除了启动网络时使用`docker-compose-couch.yaml`：
```
CHANNEL_NAME=$CHANNEL_NAME TIMEOUT=<pick_a_value> docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d
```
下面的链码**chaincode_example02**将使用CouchDB。  
*注意：如果你选择了将fabric-couchdb容器的端口映射到主机端口，请确保端口的远程访问是安全的。在开发环境下映射端口使CouchDB REST API可用，并使通过CoutchDB web接口(Fauxton)使数据库可见。在生产环境下进行端口映射要慎重，需要限制从外部访问CouchDB容器的端口。*  
你可以用**chaincode_example02**链码访问CouchDB状态数据库，就像前面讲的那样。但为了执行CouchDB特性的查询，你需要使用数据模型为JSON的链码(如**marbles02**)。你可以在`fabric/examples/chaincode/go`目录找到**marbles02**链码。
我们可以使用相同的步骤创建和加入通道，就像前面在[创建和加入通道](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#创建和加入通道)一节中讲的那样。一旦将peer加入了通道，使用下面的步骤与**marbles02**链码交互：
1. 在`peer0.org1.example.com`上安装和实例化链码:  
```
$$ peer chaincode install -n marbles -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles02
$$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org0MSP.member','Org1MSP.member')"
```
2. 创建一些弹珠并移动它们：
```
$$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
$$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble2","red","50","tom"]}'
$$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble3","blue","70","tom"]}'
$$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarble","marble2","jerry"]}'
$$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarblesBasedOnColor","blue","jerry"]}'
$$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["delete","marble1"]}'
```
3. 如果你选择在docker-compse中映射CouchDB端口，你现在就可以通过CouchDB的web接口(Fauxton)查询状态数据库，方法是打开一个浏览器并导航到下列URL：
```
http://localhost:5984/_utils
```
你可以看到一个叫`mychannel`的数据库(或你自己定义的通道名称)和里面的文档数据。
*注意：你需要更新$CHANNEL_NAME为合适的值*  
你可以通过CLI运行一般查询(如读`marble2`):
```
$$ peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["readMarble","marble2"]}'
Query Result: {"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}
```
你可以查询一个特定弹珠的历史，如`marble1`:
```
$$ peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["getHistoryForMarble","marble1"]}'
Query Result: [{"TxId":"1c3d3caf124c89f91a4c0f353723ac736c58155325f02890adebaa15e16e6464", "Valu
```
你还可以对数据内容执行富文本查询，如查询拥有者`jerry`的弹珠字段：
```
peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarblesByOwner","jerry"]}'
Query Result: [{"Key":"marble2", "Record":{"color":"red","docType":"marble","name":"marble2","owner"
```
### 为什么是CouchDB？
CouchDB是一种NoSQL解决方案。它是一个面向文档的数据库，其中文档字段被存储为键值对。字段可以是简单的键/值对，列表或映射。除了LevelDB支持的keyed/composite-key/key-range查询外，CouchDB还支持完整的富文本查询功能，例如对整个区块链数据的非键查询，因为其数据内容以JSON格式存储，完全可查询。因此，CouchDB可以满足不受LevelDB支持的许多用例的链码，审计和报告要求。

CouchDB还可以增强区块链中合规性和数据保护的安全性。因为它能够通过过滤和屏蔽事务内的属性来实现字段级别的安全性，并且在需要时授权只读权限。  

另外，CouchDB属于CAP定理的AP类型（Availability和Partition Tolerance）。它使用主 - 主复制模型。有关更多信息，请参阅CouchDB文档的“[ 最终一致性](http://docs.couchdb.org/en/latest/intro/consistency.html)”页面。但是，在每个Fabric peer下，不存在数据库副本，写入数据库将保证一致性和持久性（非`Eventual Consistency`）。

CouchDB是Fabric的第一个外部可插入状态数据库，可能也会有其他外部数据库选项。例如，IBM为关系数据库启用区块链。而CP型（一致性和分区容忍）数据库也可能是需要的，以便在没有应用级保证的情况下实现数据一致性。  

### 一个关于数据持久化的备注
如果在peer容器或CouchDB容器上需要数据持久性，有一种选择是将docker宿主机中的目录挂载到容器中的相关目录中。例如，您可以在`docker-compose-base.yaml`文件中的peer容器定义中添加以下两行：
```
volumes:
 - /var/hyperledger/peer0:/var/hyperledger/production
```
对于CouchDB容器，您可以在CouchDB容器定义中添加以下两行：、
```
volumes:
 - /var/hyperledger/couchdb0:/opt/couchdb/data
```
### Troubleshooting
- 总是干净地启动网络。使用下列命令删除共建、密钥、容器和链码镜像：
```
./byfn.sh -m down
```
*注意：如果不删除旧的容器和镜像会报错。*  
- 如果你看到Docker错误，首先检查docker版本，然后重启docker进程。docker问题往往不好识别。例如，你可能看到的错误是不能找到挂在到容器的密钥文件。  
如果你想删除镜像重新开始：
```
$ docker rm -f $(docker ps -aq)
$ docker rmi -f $(docker images -q)
```
- 如果你在创建、实例化、调用或查询命令中看到错误，确保你的通道名称和链码名称正确。
- 如果你看到下面的错误：
```
Error: Error endorsing chaincode: rpc error: code = 2 desc = Error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exits)
```
看来你已经有了一个链码镜像（例如`dev-peer1.org2.example.com-mycc-1.0`或`dev-peer0.org1.example.com-mycc-1.0`)已经在运行。删除它们重试。
```
$ docker rmi -f $(docker images | grep peer[0-9]-peer[0-9] | awk '{print $3}')
```
- 如果你看到类似下面的内容：
```
Error connecting: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure
Error: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure
```
确保你正在运行的网络是“1.0.0”镜像并且tag是“latest”。  

- 如果你看到下面的错误：
```
[configtx/tool/localconfig] Load -> CRIT 002 Error reading configuration: Unsupported Config Type ""
panic: Error reading configuration: Unsupported Config Type ""
```
说明你没有正确设置环境变量`FABRIC_CFG_PATH`。configtxgen工具需要这个变量来定位configtx.yaml。返回和执行`export FABRIC_CFG_PATH=$PWD`，然后重建你的通道工件。  
- 使用`down`选项清理网络：
```
$ ./byfn.sh -m down
```
- 如果你看到一个错误说你仍有活动的端点，则需要prune你的docker网络。这将清除之前的网络，重新启动一个干净的环境：
```
$ docker network prune
```
你会看到下列信息：
```
WARNING! This will remove all networks not used by at least one container.
Are you sure you want to continue? [y/N]
```
选择`y`。

### 理解Fabric网络
应用通过API调用智能合约。智能合约托管在网络中，靠名称和版本号识别。例如，智能合约容器的名称是`dev-peer0.org1.example.com-fabcar-1.0`，其中`fabcar`是智能合约名称，`1.0`是智能合约版本号，而`dev-peer0.org1.example.com`是peer名称。  
API可以把SDK访问。SDK封装了应用与智能合约通信的接口，如查询或接收账本更新。这些API使用几个不同的网络地址，接收一些输入参数。智能合约由peer管理员安装，然后按照链码的策略被实例化到通道中。智能合约的实例化流程与普通调用的事务流程相同，背书、排序、生效、提交，之后才能与链码容器交互（智能合约实例化就是链码容器启动）。   
#### 查询
查询是最简单的调用：一个请求和响应。最常见的查询是向状态数据库查询一个key的当前值(`GetState`)。然而，[链码shim接口](https://github.com/hyperledger/fabric/blob/release/core/chaincode/shim/interfaces.go)允许不同的Get请求，如`GetHistoryForKey`或`GetCreator`。  
创建查询需要指定一个peer、一个链码、一个通道和一系列输入(如key)和一个可用的链码函数，然后通过API`chain.queryByChaincode`发送查询到peer。相应的响应值会返回给应用客户端。  
#### 更新
账本更新开始于应用创建一个事务提议。类似于查询，创建事务请求需要指定一个peer、链码、通道、函数和一系列输入。程序之后会调用API`channel.SendTransactionProposal`发送事务提议到peer寻求背书。  
网络(也就是背书peer(可能多个))会返回一个提议响应，应用使用该响应来创建和签署事务请求。通过调用API`channel.sendTransaction`，这个事务请求被发送到排序服务。排序服务将事务捆绑入一个区块，并将它发送到通道中的所有peer以求生效(Fabcar网络只有一个peer和一个通道)。  
最后应用使用两个事件处理器API：用`eh.setPeerAddr`连接到peer的事件监听者端口，用`eh.registerTxEvent`和一个特定事务ID去注册事件。`eh.registerTxEvent`API使应用可以收到事务结果通知（就是是否生效）。  
事务流程图示参考本文的[共识过程](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E5%85%B1%E8%AF%86%E8%BF%87%E7%A8%8B)一节。  
关于事务流程的更多细节参考[Transaction Flow](http://hyperledger-fabric.readthedocs.io/en/latest/txflow.html)。  
开始链码编程参考[Chaincode for Developers](http://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html)。  
更多背书策略参考[Endorsement policies](http://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html)。  
更多fabric架构信息参考[Architecture Explained](http://hyperledger-fabric.readthedocs.io/en/latest/arch-deep-dive.html)。  
## 编写首个应用
在实践本章前，请按[启动首个网络(first-network)](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E5%90%AF%E5%8A%A8%E9%A6%96%E4%B8%AA%E7%BD%91%E7%BB%9Cfirst-network)一章的描述准备好环境。  
本章原文是官网的[Writing Your First Application](http://hyperledger-fabric.readthedocs.io/en/latest/write_first_app.html)  
本章的工作目录是`/opt/fabric-samples/fabcar`。  
在本章中，首先访问CA生成注册证书(ECert)，然后利用生成的身份(用户对象)查询和更新账本。  
### 建立开发环境
可以通过执行下列命令停止“first-network”一章中的各种容器：
```
$ cd /opt/fabric-samples/fabcar
$ ../first-network/byfn.sh -m down
```
或使用docker命令删除所有容器(可以与上面的命令混合使用)：
```
$ docker rm -f $(docker ps -aq)
```
删除所有缓存网络：
```
$ docker network prune
```
#### 安装客户端和启动网络
`/opt/fabric-samples/fabcar`目录下有个`package.json`文件，定义了示范应用的依赖包，通过执行下列命令安装依赖（与java的maven类似）：
```
$ npm install
```
使用脚本`startFabric.sh`启动网络。这个命令会启动几个Fabric实体和一个智能合约容器(链码)。
```
$ ./startFabric.sh
$ docker ps
```
用上述docker命令可以看到启动的容器清单。  

### 注册用户
#### 注册管理员用户
前面的`startFabric.sh`命令会启动一个CA容器，可以通过下列命令查看CA容器的日志：
```
$ docker logs -f ca.example.com
```
执行下列命令来注册admin用户：
```
$ node enrollAdmin.js
```
上述代码会向创建目录`hfc-key-store`，创建私钥，向CA发送证书签名请求(CSR)，然后把返回的CA签名证书(eCert)存放在`hfc-key-store`目录。  
#### 注册用户user1
admin用户是运维人员用的管理员证书。在一般的业务场景下，应用程序使用“普通用户”(非管理员)来访问Fabric网络。需要说明的是，向Fabric网络注册普通用户需要使用管理员身份，如现在就是用admin身份来注册`user1`用户。  
```
$ node registerUser.js
```
与注册admin用户类似，代码会创建user1用户私钥，向CA发送证书签名请求(CSR)，并将返回的CA签名证书(eCert)存放在`hfc-key-store`目录。  
### 查询账本
用`docker ps`命令可以看到一个`hyperledger/fabric-couchdb`的容器，这是一个保存“世界状态”的key-value数据库(couchdb)。查询账本实际上就是在查询key-value数据库中的数据（数据库变化日志保存在区块链中）。查询参数一般是一个或几个key，也可以用json串当参数进行复杂查询。  
演示查询的代码文件是`query.js`，其中可以看到下面的代码，说明应用使用`user1`作为签名实体。
```
fabric_client.getUserContext('user1', true);
```
user1的身份证明材料已经放在了`hfc-key-store`目录下，我们简单地告诉应用去获取身份。  
```
$ node query.js
```
会返回一个json串，里面是全部10辆车的信息。  
观察一下`query.js`源码。在应用的初始化区段，定义了几个变量，如通道名称、证书库地址和网络端点。
```js
var channel = fabric_client.newChannel('mychannel');
var peer = fabric_client.newPeer('grpc://localhost:7051');
channel.addPeer(peer);

var member_user = null;
var store_path = path.join(__dirname, 'hfc-key-store');
console.log('Store path:'+store_path);
var tx_id = null;
```
下面是执行查询的代码：
```js
// queryCar chaincode function - requires 1 argument, ex: args: ['CAR4'],
// queryAllCars chaincode function - requires no arguments , ex: args: [''],
const request = {
  //targets : --- letting this default to the peers assigned to the channel
  chaincodeId: 'fabcar',
  fcn: 'queryAllCars',
  args: ['']
};
```
应用执行时，它调用了peer的`fabcar`链码，执行其中的`queryAllCars`函数。 
如果想看看有哪些链码函数可以调用，可以打开文件`../chaincode/fabcar/go/fabcar.go`来看。可以看到下列函数可以调用：`initLedger`, `queryCar`, `queryAllCars`, `createCar`和`changeCarOwner`。  
下图图示了应用、智能合约和账本的关系：  
![](http://hyperledger-fabric.readthedocs.io/en/latest/_images/RunningtheSample.png)  
下面示范一下调用链码的`queryCar`函数来查询某一辆车的信息。将`query.js`中的`queryAllCars`函数换成`quieryCar`，参数key是`CAR4`。
```js
const request = {
  //targets : --- letting this default to the peers assigned to the channel
  chaincodeId: 'fabcar',
  fcn: 'queryCar',
  args: ['CAR4']
};
```
保存`query.js`，并执行，可以看到返回了`CAR4`的车辆信息：
```
$ node query.js
{"colour":"black","make":"Tesla","model":"S","owner":"Adriana"}
```
#### 更新账本
更新账本的过程：先提议，后背书，结果返回到应用，然后发送给排序节点，再写到各个peer的账本。  
![](http://hyperledger-fabric.readthedocs.io/en/latest/_images/UpdatingtheLedger.png)  
js文件`invoke.js`是更新账本的程序示范。首先可以看到程序中构造请求的代码：
```js
var request = {
  //targets: let default to the peer assigned to the client
  chaincodeId: 'fabcar',
  fcn: 'createCar',
  args: ['CAR10', 'Chevy', 'Volt', 'Red', 'Nick'],
  chainId: 'mychannel',
  txId: tx_id
};
```
在代码中通过调用链码`fabcar`的`createCar`函数，试图创建一个10号车(key是`CAR10`)。执行`invoke.js`：
```
$ node invoke.js
Store path:/opt/fabric-samples/fabcar/hfc-key-store
Successfully loaded user1 from persistence
Assigning transaction_id:  1adb925d24db20816bcfc97f0216f3b094f3291778af775c1d07d0d3179a3031
Transaction proposal was good
Successfully sent Proposal and received ProposalResponse: Status - 200, message - "OK"
info: [EventHub.js]: _connect - options {"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1}
The transaction has been committed on peer localhost:7053
Send transaction promise and event listener promise have completed
Successfully sent transaction to the orderer.
Successfully committed the change to the ledger by the peer
```
从上面可以看到关于`ProposalResponse`的终端输出和promise(一个异步调用标准)。上文的`The transaction has been committed on peer localhost:7053`表示事务已经成功写到peer。下面可以回到`query.js`，将查询条件由`CAR4`改成`CAR10`：
```js
const request = {
  //targets : --- letting this default to the peers assigned to the channel
  chaincodeId: 'fabcar',
  fcn: 'queryCar',
  args: ['CAR10']
};
```
重新执行`query.js`可以查询到最新添加的10号车的信息：
```
$ node query.js
Response is  {"colour":"Red","make":"Chevy","model":"Volt","owner":"Nick"}
```
下面重新修改代码`invoke.js`。将函数`createCar`改成`changeCarOwner`，目的是把10号车的车主改成`Dave`：
```js
var request = {
  //targets: let default to the peer assigned to the client
  chaincodeId: 'fabcar',
  fcn: 'changeCarOwner',
  args: ['CAR10', 'Dave'],
  chainId: 'mychannel',
  txId: tx_id
};
```
重新执行`invoke.js`和`query.js`，可以看到车主信息被从`Nick`改成了`Dave`。
```
$ node invoke.js
$ node query.js
Response is  {"colour":"Red","make":"Chevy","model":"Volt","owner":"Dave"}
```
本章主要讲应用开发，后面的章节会讲链码的开发。  

## 链码教程
[官方原文](http://hyperledger-fabric.readthedocs.io/en/latest/chaincode.html)  
链码是一个程序，用Go、Node.js编写(未来会支持其他语言，如Java)，实现了一个规定的接口。链码运行在安全的Docker容器中，隔离于背书peer过程。链码通过应用提交的事务来初始化和管理账本状态。  
链码处理网络成员都同意的业务逻辑，所以它可以被认为是“智能合约”。链码创建的状态是不能直接被其他链码访问的（scoped）。但在同一个网络内（一般指通道？），通过适当的授权，一个链码可以调用其它链码而从访问它的状态。  
我们提供了观看链码的两个不同视角，一个面向开发链码应用的开发者，另一个面向区块链网络的管理者，他会利用Hyperledger Fabric API去安装、实例化和更新链码，但不会涉及链码应用开发（[链码教程:链码运维](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E9%93%BE%E7%A0%81%E6%95%99%E7%A8%8B%E9%93%BE%E7%A0%81%E8%BF%90%E7%BB%B4)）。  

## 链码教程:链码开发
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html)    
每个链码程序都必须实现`Chaincode`[接口](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#Chaincode)。下面是Go语言的`Chaincode`接口：
```go
type Chaincode interface {
    // Init is called during Instantiate transaction after the chaincode container
    // has been established for the first time, allowing the chaincode to
    // initialize its internal data
    Init(stub ChaincodeStubInterface) pb.Response

    // Invoke is called to update or query the ledger in a proposal transaction.
    // Updated state variables are not committed to the ledger until the
    // transaction is committed.
    Invoke(stub ChaincodeStubInterface) pb.Response
}
```
Fabric通过调用这些约定的函数来运行事务。在响应中通过调用这些方法来接收事务。当链码收到`instantiate`或`upgrade`事务时，`Init`方法会被调用，使链码可以执行必要的初始化，包括应用状态初始化。在响应中收到`invoke`事务时`Invoke`方法会被调用，使链码可以处理事务提议(proposal)。  
链码的[“shim”](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim)API中的另一个接口是`ChaincodeStub`[接口](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub))。它用于访问和修改账本，以及链码间的调用。  
在这个教程中，我们将示范使用这些API实现一个简单链码应用，来管理简单“资产”。   

### 简单资产链码
我们的应用是一个基本示范链码，用来在账本中创建资产(键值对)。  
假定`GOPATH=/opt/gopath`，下面创建示范代码的工作目录：
```
$ mkdir -p $GOPATH/src/sacc && cd $GOPATH/src/sacc
$ touch sacc.go
```
touch命令创建了一个叫sacc.go的空文件。下面描写如何向sacc.go中添加代码。    
首先用go的import语句添加链码的必要依赖，shim包和[peer protobuf](http://godoc.org/github.com/hyperledger/fabric/protos/peer)包。然后，增加一个`SimpleAsset`结构作为链码shim函数的接收者。
```go
package main

import (
    "fmt"

    "github.com/hyperledger/fabric/core/chaincode/shim"
    "github.com/hyperledger/fabric/protos/peer"
)

// SimpleAsset implements a simple chaincode to manage an asset
type SimpleAsset struct {
}
```
实现`Init`函数：
```go
// Init is called during chaincode instantiation to initialize any data.
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {

}
```
#### 初始化
注意，链码的程序版本更新也会调用`Init`函数。（在Fabric中链码程序更新也是通过事务，同不同事务一样，所以会调用`Init`函数。）  

下面，我们调用[ChaincodeStubInterface.GetStringArgs](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.GetStringArgs)函数取得调用`Init`的参数，并进行验证。在我们案例中，仅仅验证一下参数是否为键值对。  
```go
// Init is called during chaincode instantiation to initialize any
// data. Note that chaincode upgrade also calls this function to reset
// or to migrate data, so be careful to avoid a scenario where you
// inadvertently clobber your ledger's data!
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
  // Get the args from the transaction proposal
  args := stub.GetStringArgs()
  if len(args) != 2 {
    return shim.Error("Incorrect arguments. Expecting a key and a value")
  }
}
```
调用生效证后，我们将初始状态保存到账本。为此，我们用传入的键值对当参数，调用[ChaincodeStubInterface.PutState](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.PutState)。假如一切正常，返回一个peer.Response对象来表明初始化成功。  
```go
// Init is called during chaincode instantiation to initialize any
// data. Note that chaincode upgrade also calls this function to reset
// or to migrate data, so be careful to avoid a scenario where you
// inadvertently clobber your ledger's data!
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
  // Get the args from the transaction proposal
  args := stub.GetStringArgs()
  if len(args) != 2 {
    return shim.Error("Incorrect arguments. Expecting a key and a value")
  }

  // Set up any variables or assets here by calling stub.PutState()

  // We store the key and the value on the ledger
  err := stub.PutState(args[0], []byte(args[1]))
  if err != nil {
    return shim.Error(fmt.Sprintf("Failed to create asset: %s", args[0]))
  }
  return shim.Success(nil)
}
```
#### 调用链码
首先，添加Invoke函数：
```go
// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The 'set'
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {

}
```
就像上面的`Init`函数，我们需要从`ChaincodeStubInterface`中取出参数。`Invoke`函数的参数将会是链码应用的函数名。在我们的案例中，应用只有两个函数：`set`和`get`，用来设置资产值或取资产的当前状态。我们首先调用[ChaincodeStubInterface.GetFunctionAndParameters](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.GetFunctionAndParameters)来获取链码应用的函数名和参数。   
```go
// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The Set
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    // Extract the function and args from the transaction proposal
    fn, args := stub.GetFunctionAndParameters()

}
```
然后，我们验证函数名是`set`或`get`，调用这些链码应用函数，通过`shim.Success`或`shim.Error`返回相应的响应，它们会将响应串行化为gRPC protobuf消息。  
```go
// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The Set
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    // Extract the function and args from the transaction proposal
    fn, args := stub.GetFunctionAndParameters()

    var result string
    var err error
    if fn == "set" {
            result, err = set(stub, args)
    } else {
            result, err = get(stub, args)
    }
    if err != nil {
            return shim.Error(err.Error())
    }

    // Return the result as success payload
    return shim.Success([]byte(result))
}
```
#### 实现链码应用
正如上面指出的那样，我们的链码应用需要实现两个函数，它们会被`Invoke`函数调用。为了访问账本状态，我们会用到链码shim API的[ChaincodeStubInterface.PutState](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.PutState)和[ChaincodeStubInterface.GetState](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.GetState)函数。  
```go
// Set stores the asset (both key and value) on the ledger. If the key exists,
// it will override the value with the new one
func set(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 2 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
    }

    err := stub.PutState(args[0], []byte(args[1]))
    if err != nil {
            return "", fmt.Errorf("Failed to set asset: %s", args[0])
    }
    return args[1], nil
}

// Get returns the value of the specified asset key
func get(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 1 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key")
    }

    value, err := stub.GetState(args[0])
    if err != nil {
            return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
    }
    if value == nil {
            return "", fmt.Errorf("Asset not found: %s", args[0])
    }
    return string(value), nil
}
```
#### 放在一起
最后，需要添加`main`函数，它会调用[shim.Start](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#Start)函数。下面是链码的完整源码：  
```go
package main

import (
    "fmt"

    "github.com/hyperledger/fabric/core/chaincode/shim"
    "github.com/hyperledger/fabric/protos/peer"
)

// SimpleAsset implements a simple chaincode to manage an asset
type SimpleAsset struct {
}

// Init is called during chaincode instantiation to initialize any
// data. Note that chaincode upgrade also calls this function to reset
// or to migrate data.
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
    // Get the args from the transaction proposal
    args := stub.GetStringArgs()
    if len(args) != 2 {
            return shim.Error("Incorrect arguments. Expecting a key and a value")
    }

    // Set up any variables or assets here by calling stub.PutState()

    // We store the key and the value on the ledger
    err := stub.PutState(args[0], []byte(args[1]))
    if err != nil {
            return shim.Error(fmt.Sprintf("Failed to create asset: %s", args[0]))
    }
    return shim.Success(nil)
}

// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The Set
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    // Extract the function and args from the transaction proposal
    fn, args := stub.GetFunctionAndParameters()

    var result string
    var err error
    if fn == "set" {
            result, err = set(stub, args)
    } else { // assume 'get' even if fn is nil
            result, err = get(stub, args)
    }
    if err != nil {
            return shim.Error(err.Error())
    }

    // Return the result as success payload
    return shim.Success([]byte(result))
}

// Set stores the asset (both key and value) on the ledger. If the key exists,
// it will override the value with the new one
func set(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 2 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
    }

    err := stub.PutState(args[0], []byte(args[1]))
    if err != nil {
            return "", fmt.Errorf("Failed to set asset: %s", args[0])
    }
    return args[1], nil
}

// Get returns the value of the specified asset key
func get(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 1 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key")
    }

    value, err := stub.GetState(args[0])
    if err != nil {
            return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
    }
    if value == nil {
            return "", fmt.Errorf("Asset not found: %s", args[0])
    }
    return string(value), nil
}

// main function starts up the chaincode in the container during instantiate
func main() {
    if err := shim.Start(new(SimpleAsset)); err != nil {
            fmt.Printf("Error starting SimpleAsset chaincode: %s", err)
    }
}
```
#### 构建链码
编译链码：
```
$ go get -u --tags nopkcs11 github.com/hyperledger/fabric/core/chaincode/shim
$ go build --tags nopkcs11
```
执行上述`go get`时碰到问题：
```
# cd /opt/gopath/src/github.com/hyperledger/fabric; git pull --ff-only
error: Your local changes to the following files would be overwritten by merge:
        docs/source/kafka.rst
Please, commit your changes or stash them before you can merge.
```
解决办法是：
```
$ cd /opt/gopath/src/github.com/hyperledger/fabric;
$ git reset --hard
```
上述命令的含义是放弃本地的修改，以网上版本为准。  
回到原来的目录，重新执行`go get`，问题解决。  
#### 使用开发模式测试
通常链码由peer启动和维护。但在“开发模式”下，链码由用户构建和启动。在快速代码/构建/运行/调试周期转换的链码开发阶段，此模式非常有用。  
我们利用预生成的排序器和通道工件启动“开发模式”，获得一个示范开发网络。用户可以直接跳到编译链码过程和驱动调用。  

### 调试与测试
需要启动3个终端窗口。  
#### 窗口1：启动网络
```
$ cd /opt/fabric-samples/chaincode-docker-devmode
$ docker-compose -f docker-compose-simple.yaml up
```
上面的命令用`SingleSampleMSPSolo`排序器profile启动网络，并以开发模式启动peer。还启动了两个附加容器，一个是链码环境，另一个是与链码交互的CLI。创建和加入通道的命令被嵌入到CLI容器中，从而我们可以直接调用链码。  
#### 窗口2：构建和启动链码
用下列命令进入`chaincode`容器内的bash环境:
```
$ docker exec -it chaincode bash
root@d2629980e76b:/opt/gopath/src/chaincode#
```
虽然看上去与宿主机类似，其实已经在`chaincode`容器内。下面执行的都是该容器内的命令。下面编译你的链码：  
```
$ cd sacc
$ go build
```
运行链码：
```
$ CORE_PEER_ADDRESS=peer:7051 CORE_CHAINCODE_ID_NAME=mycc:0 ./sacc
```
上述链码随peer启动，链码日志表示它成功注册到peer。注意，现阶段的链码还没有与通道关联。这个会在后续的步骤中通过实例化命令做到。  
#### 窗口3：使用链码
虽然你处于`--peer-chaincodedev`模式，你仍然需要安装链码，因此生命周期系统链码可以像通常一样执行检查。将来这个需求可能在`--peer-chaincodedev`模式下移除。  
我们利用CLI容器执行这些调用：
```
$ docker exec -it cli bash
$ peer chaincode install -p chaincodedev/chaincode/sacc -n mycc -v 0
$ peer chaincode instantiate -n mycc -v 0 -c '{"Args":["a","10"]}' -C myc
```
现在发出一个调用将`a`的值改成`20`。
```
$ peer chaincode invoke -n mycc -c '{"Args":["set", "a", "20"]}' -C myc
```
最后，查询`a`，可以看到值是`20`。
```
$ peer chaincode query -n mycc -c '{"Args":["query","a"]}' -C myc
```
#### 测试新链码
默认我们仅挂载`sacc`。然而，你通过将它们添加到`chaincode`子目录来测试不同的链码（需要重启网络）。你可以在`chaincode`容器中访问它们。  

#### 链码加密
在某些情况下，需要对key关联的全部值或部分值进行加密。例如，当一个人的社会安全号码或地址在写入账本时，可能不希望这些数据以明文形式出现。链码加密利用[实体扩展](https://github.com/hyperledger/fabric/tree/master/core/chaincode/shim/ext/entities)，它内部包装了BCCSP，支持加密和椭圆曲线数字签名。例如，为了加密，链码的调用者通过临时字段传入加密密钥。然后可以将相同的密钥用于随后的查询操作，从而允许对加密的状态值进行解密。

对于更多详细信息和示范，看`fabric/examples`目录下的[Encc Example](https://github.com/hyperledger/fabric/tree/master/examples/chaincode/go/enccc_example)。特别注意utils.go 助手程序。该实用程序加载链码代码API和实体扩展，并构建一个新的函数类（例如`encryptAndPutState`＆`getStateAndDecrypt`），以便示例加密链码利用。因此，链码现在可以和基本的shim API结合起来，使`Get`和`Put`可以添加解密和加密功能。  

## 链码教程:链码运维
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/chaincode4noah.html)  
链码是一个程序，用Go、Node.js编写(未来会支持其他语言，如Java)，实现了一个规定的接口。链码运行在安全的Docker容器中，隔离于背书peer过程。链码通过应用提交的事务来初始化和管理账本状态。  
链码处理网络成员都同意的业务逻辑，所以它可以被认为是“智能合约”。链码创建的状态是不能直接被其他链码访问的（scoped）。但在同一个网络内（一般指通道？），通过适当的授权，一个链码可以调用其它链码而从访问它的状态。  
本章假定了一个叫诺亚的运维工程师，通过他的视角关注链码。根据诺亚的喜好，我们专注于链码的全生命周期维护，即包装、安装、实例化和升级链码的过程。  
### 链码生命周期
Hyperledger Fabric API允许与区块链网络中的不同节点(peer、orderer和MSP)交互，它还允许在背书peer节点上打包、安装、实例化和升级链码。Hyperledger Fabric各语言SDK对Hyperledger Fabric API进行抽象以利于应用开发，所以它可以用于管理链码生命周期。此外，Hyperledger Fabric API还可以通过CLI直接访问，这在本章我们会用到。  
我们提供了四个命令去管理链码生命周期：`package`、`install`、`instantiate`和`upgrade`。在未来的版本中，我们正考虑增加`stop`和`start`事务去禁用和重新启用链码，而不用实际卸载它。在链码被成功安装和实例化后，链码是活跃状态(运行中)，可以通过 `invoke`事务处理事务。链码可以在安装后多次升级（版本更新）。  
### 打包
链码包由三部分组成：
- 链码，就象在`ChaincodeDeploymentSpec`(简称CDS)中定义的。CDS通过code和其它属性(如名称和版本)来定义链码包  
- 一个可选的实例化策略，这有时被称为背书策略  
- 链码“拥有者”（实体）的一组数字签名  

签名用于以下目的：  
- 建立链码的所有权  
- 允许验证包裹的内容  
- 允许检测包裹篡改  
链码实例化事务的创建者需要通过链码的实例化策略的验证。  

#### 建包
有两个方法对链码打包，复杂的和简单的。当链码具有多个拥有者时，它需要被多个身份签名。这需要我们首先建立一个签名的链码包(`SignedCDS`)，然后发给其他拥有者进行签名。这是一个复杂流程。
简化流程是，当你部署的SignedCDS只有一个签名，而且签名者就是安装事务的发起者。（即安装一个自己签名的包到自己的peer）  
先讲复杂流程。  
创建一个签名的链码包，适用下列命令：  
```
$ peer chaincode package -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -v 0 -s -S -i "AND('OrgA.admin')" ccpack.out
```
`-s`选项表示创建一个多拥有者签名的包，如果不加就简单创建一个纯CDS。当指定了`-s`选项，如果有其他拥有者需要签名，则`-S` 选项必须设置。否则，会创建一个仅包含实例化策略的SignedCDS。  
`-S`选项使处理流程使用 `core.yaml`文件中`localMspid`属性下定义的MSP身份对包进行签名。  
`-S`选项使可选的。但如果一个包没有签名，它就不能被其他拥有者使用`signpackage`命令进行签名。  
`-i`选项用于为链码指定实例化策略。实例化策略与背书策略的格式相同，都是指定哪些身份可以实例化这个链码。在上面的例子中，只有`OrgA`的管理员(admin)可以实例化这个链码。如果没有设置策略，会使用默认策略，则只允许peer的MSP的管理员身份去实例化链码。  
#### 包签名
一个创建时签名的链码包可以被移交给其他拥有者查看和签名。流程支持out-of-band对链码包签名。  

[ChaincodeDeploymentSpec](https://github.com/hyperledger/fabric/blob/master/protos/peer/chaincode.proto#L78)可以选择被集体拥有者签名，而从创建一个[SignedChaincodeDeploymentSpec](https://github.com/hyperledger/fabric/blob/master/protos/peer/signed_cc_dep_spec.proto#L26)(或叫SignedCDS)。SignedCDS包含3个元素：  
 1. CDS包含的链码源码、名称和版本号。  
 2. 一个链码的实例化策略，表述为背书策略。  
 3. 链码拥有者列表，通过[背书](https://github.com/hyperledger/fabric/blob/master/protos/peer/proposal_response.proto#L111)定义。    

*【注意】：当链码在一些通道实例化时，这个背书策略通过out-of-band确定MSP身份。如果实例化策略没有指定，默认策略是通道的任何MSP管理员。*  

每个拥有者都对`ChaincodeDeploymentSpec`进行背书，背书方法是对CDS与拥有者身份（如证书）的组合结果进行签名(算法：sign(ProposalResponse.payload + endorser))。  
一个链码拥有者使用下面的命令对以前创建的签名包进行签名：
```
$ peer chaincode signpackage ccpack.out signedccpack.out
```
`ccpack.out`和`signedccpack.out`分别是输入包和输出包。`signedccpack.out`中包含了一个对包的新增签名，签名使用了本地MSP。  

#### 安装链码
安装(`install`)事务按规定格式对链码的源码进行打包，这个格式称为`ChaincodeDeploymentSpec`（或称CDS），该事务将链码安装在将来要运行它的peer节点上。  

*【注意】：你必须将链码安装在要运行链码的通道的每个背书peer节点上。*  

当`install`API简单给予了一个`ChaincodeDeploymentSpec`，它将使用默认实例化策略和包含一个空的拥有者列表。  

*【注意】：为了保证链码逻辑对网络上的其他成员保密，链码只安装在链码拥有者的背书peer节点上（可能存在一个或多个拥有者）。哪些没有链码的成员，不能是链码事务的背书者；也就是说，他们不能执行链码。然而，他们仍然可以生效和提交事务到账本。*  

为了安装链码，发送一个[SignedProposal](https://github.com/hyperledger/fabric/blob/master/protos/peer/proposal.proto#L104)到`lifecycle system chaincode`(LSCC)(LSCC会在[系统链码](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E7%B3%BB%E7%BB%9F%E9%93%BE%E7%A0%81)一节中描述)。例如，使用CLI安装**sacc**示范链码（前文在“链码教程:链码开发-[调试与测试](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E8%B0%83%E8%AF%95%E4%B8%8E%E6%B5%8B%E8%AF%95)”一节中描述过）的命令如下：
```
$ peer chaincode install -p chaincodedev/chaincode/sacc -n mycc -v 0
```
CLI内部为**sacc**创建一个`SignedChaincodeDeploymentSpec`，并发送它到本地peer，peer调用LSCC上的`Install`方法。`-p`选项指定了链码的路径，它必须位于用户`GOPATH`的源码树上，如`$GOPATH/src/sacc`。[CLI](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#cli)一节有这个命令选项的详细描述。  
请注意，为了安装在peer上，SignedProposal的签名必须来自peer的本地MSP管理员之一。  

#### 实例化
`instantiate`事务调用`lifecycle System Chaincode`(LSCC)在一个通道上创建和实例化某个链码。这是一个链码-通道绑定过程：一个链码可以绑定到任意数量的通道，独立和互不依赖地运行在每个通道上。换句话说，无论链码在多少个其他通道上安装和实例化，对于提交事务的通道状态是隔离的。  
`instantiate`事务的创建者必须满足包含在SignedCDS中的链码实例化策略，必须还是通道的写入者（这是通道创建时的配置之一）。这对于通道安全很重要，可以阻止恶意实体部署链码和欺骗成员执行非绑定通道的链码。  
例如，回想一下，默认实例化策略是任何通道MSP管理员，因此链码实例化事务的创建者必须是通道管理员的成员。交易提议到达背书者时，会根据实例化策略验证创建者的签名。在提交到账本之前，在交易生效期间再次执行此操作。  
实例化事务还为通道上的链码建立了背书策略。背书策略描述了事务结果可以被通道成员接受的证据需求。  
例如，使用CLI实例化**sacc**链码和用`john`和`0`初始化状态，命令如下：
```
$ peer chaincode instantiate -n sacc -v 1.0 -c '{"Args":["john","0"]}' -P "OR ('Org1.member','Org2.member')"
```
*【注意】上面的背书策略(CLI使用波兰语表示法)，所有的**sacc**事务需要一个Org1成员或Org2成员的背书。就是说，为了事务生效，Org1或Org2需要对调用(Invoke)**sacc**的执行结果签名。*   
实例化成功后，通道中的链码进入活动状态，准备好处理任意[ENDORSER_TRANSACTION](https://github.com/hyperledger/fabric/blob/master/protos/common/common.proto#L42)类型的事务提议。当事务到达背书peer时，它们会被并发处理。  
#### 版本更新
链码可以在任何时间更新版本，版本是SignedCDS的组成部分。SignedCDS的其它部分，如拥有者和实例化策略是可选项。然而，链码名称必须相同，否则它会被视为完全不同的链码。  
版本更新前，链码的新版本必须已经在背书者节点上安装。更新是一个类似于实例化的事务，它绑定新版本的链码到通道。绑定链码旧版本的通道仍然运行旧版本。话句话说，`upgrade`事务仅影响提交了更新事务的通道。  

*【注意】，由于链码的多个版本可能同时有效，更新过程不会自动删除就版本，因此用户必须临时管理它。*  

更新事务还是与`instantiate`事务由细微的不同：`upgrade`事务检查当前链码实例化策略，不是新策略(如果指定了策略)。这确保了只有在当前实例化策略中存在的成员才可以更新链码。  

*【注意】，在更新时，链码的`Init`函数将被调用去执行相关数据更新或重新初始化，所以链码更新时要小心避免重置状态。*  

#### 停止和启动
注意`stop`和`start`生命周期事务还没有被实现。然而，你可以手工停止链码，办法是从每个背书者peer删除链码容器和SingedCDS包。在每个运行背书peer节点的主机或虚机上删除链码容器，然后删除SignedCDS。  
```
(注意，为了从peer节点删除CDS，你需要先进入peer节点的容器。我们提供了干这个的工具脚本)
$ docker rm -f <container id>
$ rm /var/hyperledger/production/chaincodes/<ccname>:<ccversion>
```
停止在用于以受控方式进行升级的工作流程中是有用的，其中链码可以在发布升级之前在所有peer的信道上停止。  
#### CLI

*【注意】：我们正在评估是否发布平台专属Hyperledger Fabric peer二进制包。在此之前，你可以在一个docker容器中简单调用命令。*  

为了显示当前可用的CLI命令，在运行中的`fabric-peer`Docker容器中执行下列命令：
```
$ docker run -it hyperledger/fabric-peer bash
(peer chaincode --help)
```
它将显示类似的以下输出：
```
Usage:
  peer chaincode [command]

Available Commands:
  install     Package the specified chaincode into a deployment spec and save it on the peer's path.
  instantiate Deploy the specified chaincode to the network.
  invoke      Invoke the specified chaincode.
  list        Get the instantiated chaincodes on a channel or installed chaincodes on a peer.
  package     Package the specified chaincode into a deployment spec.
  query       Query using the specified chaincode.
  signpackage Sign the specified chaincode package
  upgrade     Upgrade chaincode.

Flags:
    --cafile string      Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
-h, --help               help for chaincode
-o, --orderer string     Ordering service endpoint
    --tls                Use TLS when communicating with the orderer endpoint
    --transient string   Transient map of arguments in JSON encoding
```
为了方便在脚本式应用中使用，`peer`命令在失败事件中总是产生非零的返回码。  
链码命令的例子：  
```
peer chaincode install -n mycc -v 0 -p path/to/my/chaincode/v0
peer chaincode instantiate -n mycc -v 0 -c '{"Args":["a", "b", "c"]}' -C mychannel
peer chaincode install -n mycc -v 1 -p path/to/my/chaincode/v1
peer chaincode upgrade -n mycc -v 1 -c '{"Args":["d", "e", "f"]}' -C mychannel
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","e"]}'
peer chaincode invoke -o orderer.example.com:7050  --tls --cafile $ORDERER_CA -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}'
```
### 系统链码
系统链码与普通链码具有相同的编程模型，知识它运行在peer进程中，而不是在隔离的容器中。因此，系统链码构建在peer可执行文件中，它不会遵循上述同样的生命周期。特别是，安装、实例化和版本更新不会用在系统链码上。  
系统链码的目的是减少peer和链码间gRPC通信成本，和管理灵活性的折中。例如，系统链码只能与peer程序一起更新。它必须以一套固定参数注册，且不能有背书策略或背书策略函数。  
Hyperledger Fabric中使用系统链代码来实现许多系统行为，以便系统集成商可以根据需要替换或修改它们。  
当前系统链码的列表：  
1. [LSCC](https://github.com/hyperledger/fabric/tree/master/core/scc/lscc) 生命周期系统链码，处理上述的生命周期请求。  
2. [CSCC](https://github.com/hyperledger/fabric/tree/master/core/scc/cscc) 配置系统链码，处理peer端的通道配置。  
3. [QSCC](https://github.com/hyperledger/fabric/tree/master/core/scc/qscc) 查询系统链码，提供账本查询API，例如获取区块和交易。  
4. [ESCC](https://github.com/hyperledger/fabric/tree/master/core/scc/escc) 背书系统链码，通过签署交易提议响应来处理背书。  
5. [VSCC](https://github.com/hyperledger/fabric/tree/master/core/scc/vscc) 生效系统链码，处理事务生效，包括检查背书策略和多版本并发控制。  

更改或覆盖这些系统链码要小心，特别是LSCC、ESCC和VSCC，因为它们处在主事务的运行路径上。值得注意的是，VSCC将区块将提交到账本之前的生效验证，通道中的所有peer计算相同的生效验证以避免账本分歧（非确定性）是很重要的。因此，如果修改或更换VSCC，需要特别小心。  


### 密钥生成器
我们用`cryptogen`工具为不同的网络实体生成密码学文件。这些证书表达身份，对实体间通信和交易认证进行签名和验证。  
Cryptogen的配置文件是`crypto-config.yaml`，该文件包括网络拓扑，允许我们为组织以及属于组织的组件生成一系列证书和密钥。每个组织都会分配一个根证书(`ca-cert`)，该证书绑定特殊组件(peer和orderer)到组织。  
配置文件中有个`count`变量，我们用它来指定组织下的peer数量，在我们示例中，每个组织下有两个peer。我们在本文不会详述[X509证书和PKI](https://en.wikipedia.org/wiki/Public_key_infrastructure)。  
在`crypto-config.yaml`文件中，注意`OrdererOrgs`之下的“Name”, “Domain” and “Specs”参数。网络实体的命名约定是：`{{.Hostname}}.{{.Domain}}`。例如，排序节点的名称是`orderer.example.com`，这关联了一个MSP ID `Orderer`，关于MSP的更多细节参考[Membership Service Providers (MSP)](http://hyperledger-fabric.readthedocs.io/en/latest/msp.html)文档。  
执行`cryptogen`后，生成的证书和密钥保存到了目录`crypto-config`下。  
#### 配置交易生成器
工具`configtxgen`用于生成四个配置工件：
- 排序器(orderer)`genesis block`  
- 通道`configuration transaction`  
- 两个`anchor peer transactions`，每个组织生成一个  
关于`configtxgen`更多细节参考[Channel Configuration (configtxgen)](http://hyperledger-fabric.readthedocs.io/en/latest/configtxgen.html)。  

# Fabric运维
参考另一篇wiki文章《[Fabric运维](https://github.com/wbwangk/wbwangk.github.io/wiki/Fabric%E8%BF%90%E7%BB%B4)》  


### 共识过程(视频截图)
[出处](https://developer.ibm.com/courses/all/ibm-blockchain-foundation-developer/?course=begin#12034)   
- Committing Peer  
  Maintains Ledger and state. Commits Transactions. May hold smart contract(chaincode).  
- Endorsing Peer  
  Specialized committing peer that receives a transaction proposal for endorsement, responds granting or denying endorsement. Must hold smart contract.  
- Ordering Nodes(service)  
  Approves the inclusion of transaction blocks into the ledger and communicates with committing and endorsing peer nodes. Does not hold smart contract. Does not hold ledger.  
step 1/7 - propose transaction:  
![](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/images/fabric_transaction.jpg)  
step 2/7 - execute transaction:  
![](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/images/fabric_transaction2.jpg)
step 3/7 - propose response:  
![](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/images/fabric_transaction3.jpg)  
step 4/7 - order transaction:  
![](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/images/fabric_transaction4.jpg)  
step 5/7 - deliver transaction: 
![](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/images/fabric_transaction5.jpg)  
step 6/7 - validate transaction: 
![](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/images/fabric_transaction6.jpg)  
step 7/7 - notify transaction: 
![](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/images/fabric_transaction7.jpg)  

Transactions typically follow a seven-step process to become a block on the chain.
## 术语
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/glossary.html)  

#### 锚点Peer
它是通道中的一个peer节点，其他peer可以发现它并与它通信。通道中的每个[成员](#成员)都一个锚点peer(或者多个锚点防范单点失效)，用于通道中属于不同成员的peer可以发现全部peer。  

#### 区块
一组有序的事务，通过哈希链接到通道中的前一个区块。

#### 链
账本的链式一个事务日志，组织成多个哈希值链接的区块。peer从排序服务接收事务区块，根据背书策略和并发违规将区块中的事务标记为生效和失效，而且在peer的文件系统中将区块追加到哈希链。  

#### 链码
链码是运行在一个账本上的软件，编码了资产和事务指令（业务逻辑）来修改资产。  

#### 通道
通道是一个私有区块链，它允许数据隔离和保密。通道的账本被通道中所有peer共享，与通道交互的事务参与方必须被通道正确地认证。通道被[配置区块](#配置区块)定义。  

#### 提交(Commitemnt)
通道中的每个[Peer](#Peer)生效事务的有序区块，然后提交(写/追加)区块到它的通道[账本](#账本)副本。Peer会标记每个区块的每个事务为生效或失效。  

#### 并发控制版本检查
这是一个保持通道内多个peer之间状态同步的方法。peer并行执行事务，在提交到账本前，peer用读检测数据在执行的时候没有被改变。如果事务的读数据在执行时和提交时之间被改变了，则发生了并发控制版本检查违规，事务在账本中被标记为失效，不会改变状态数据库的数据。  

#### 配置区块
为系统链(排序服务)或通道容纳成员定义和策略配置数据。任何对一个通道或全部网络(如成员离开或加入)的任何配置修改都导致一个新的配置区块追加到相应链中。配置区块包含创世区块，再加上delta。

#### 共识
贯穿整个事务流程的一个更广泛的术语，用来按顺序生成一致性并确认按一组事务构组成区块的正确性。  

#### 当前状态
账本的当前状态表示包含在它的链式交易日志中的所有密钥的最新值。针对包含在已处理区块中的每个有效事务，peer将最新值提交到账本的当前状态。由于当前状态代表通道已知的所有最新键值，因此有时称为世界状态。链码针对当前状态数据执行事务提议。  

#### 动态成员资格
Hyperledger Fabric在不影响这个网络的可靠性的前提下，支持增减会员、peer和排序服务节点。当业务关系调整和引各种原因需要增减实体时，动态成员资格是很需要的。   

#### 背书
指一个过程：特定peer节点执行一个链码事务，返回提议响应到客户端应用。提议响应包含链码执行相应信息、结果(read set和write set)、事件和一个签名（证明peer执行了链码）。链码应用程序遵照背书策略，其中规定了背书peer。  

#### 背书策略
定义了通道中的哪些peer节点必须执行附加在链码应用中的事务，和需要的响应(背书)组合。需要这样一个策略，定义一个事务至少得到几个peer的同意，或最少多少比例的peer同意，或全部peer的同意，这个策略会附加到一个特定链码应用上。策略可以根据应用程序和期望水平来反对不当行为（有意或无意）。一个事务在被提交peer标记为生效前，必须符合背书策略。对部署、实例化事务也需要一个明确的背书政策。  

#### Hyperledger Fabric CA
Hyperledger Fabric CA是向网络成员组织和它们的用户发放PKI证书的默认CA组件。CA向每个成员发放一个根证书(rootCert)，象每个授权用户发放一个登记证书(ECert)。  

#### 创世区块
初始化区块链网络或通道的配置区块，也作为链上的第一个区块。  

#### Gossip协议
gossip数据广播协议完成三个功能：1）管理peer发现和通道成员资格；2）在通道中所有peer之间广播账本数据；3）在通道中所有peer之间同步账本状态。更多细节参考[Gossip](https://hyperledger-fabric.readthedocs.io/en/latest/gossip.html)主题。  

#### 初始化
一个初始化链码应用的方法。  

#### 安装
将链码放置到peer的文件系统的过程。

#### 调用(invoke)
用于调用链码函数。一个客户端应用通过发送事务提议到一个peer来调用链码。peer会执行链码和返回一个背书提议响应到客户端应用。客户端应用收集足够的提议响应以便满足背书策略，然后递交事务结果以便排序、生效和提交。客户端应用程序也可以选择不去递交事务结果。例如查询账本，客户端应用程序通常不会递交这个只读事务，除非是为了审计的目的想把读账本的行为记录日志。调用包括通道id、调用的链码函数和参数数组。  

#### 领导peer
每个[成员](#成员)在它订阅的每个通道中都可以有多个peer。其中一个peer可以作为该通道的领导peer，代表成员与网络排序服务进行通信。排序服务“递送”区块到一个通道的领导peer(可能多个)，它(或它们)再分发区块到成员集群的其他peer。  

#### 账本
账本是一个通道链和被通道中所有peer共同维护的当前状态数据。  

#### 成员
一个在网络中拥有特定根证书的独立法律实体。象peer和应用客户端这样的网络组件会被关联到成员。  

#### 成员服务提供商
成员服务提供商（MSP）是指为客户端提供（匿名）凭证的系统的抽象组件，以及参与Hyperledger/fabric网络的peer。客户端使用这些凭证对其事务进行身份验证，peer使用这些凭据来验证事务处理结果（背书）。在与系统的事务处理组件紧密连接的同时，这个接口的目标是定义成员服务组件，这样可以顺利地插入替代的实现，而无需修改系统的事务处理组件的核心。  

#### 成员服务
成员服务在授权区块链网络上认证、授权和管理身份。在peer和orderer节点上运行的成员服务代码认证和授权区块链操作。它是个成员服务提供商抽象的一个基于PKI的实现。

#### 排序服务
一个将事务排序进入区块的节点集合。排序独立于peer过程而存在，并以先来先服务的原则为网络上的所有通道进行事务排序。
一个集中或非几种的服务，对区块中事务进行排序。您可以选择“排序”功能的不同实现方式 - 例如：简化和测试的“solo”，用于碰撞容错的Kafka，或用于拜占庭容错的sBFT/PBFT。您也可以开发自己的协议来插入服务。排序服务在开箱即用的SOLO和Kafa实现的之外支持插件式实现。排序服务一般绑定到整个网络；它存放了每个[成员](#成员)的密钥身份材料。    

#### Peer
peer是一个网络实体，负责维护账本和为了读写账本而运行链码容器。peer被成员拥有和维护。  

#### 策略
是背书、生效、链码管理和网络/通道管理的策略。  

#### 提议
针对通道中的某个peer发出的背书请求。提议要么是个实例化请求，要么是个调用（读或写）请求。

#### 查询
查询是一个链码调用，它读账本当前状态但不会写账本。链码函数可以对账本按key查询，或按一组key查询。由于查询不会修改账本状态，客户端应用程序通常不会为了排序、验证和提交而递交只读事务。偶尔，客户端应用选择为了排序、验证和提交而递交只读事务，例如如果客户端需要账本在某个时间点的当前状态的审计证明。  

#### SDK
Hyperledger Fabric客户端SDK是为了方便开发者编写和测试链码应用而提供的一个结构化库环境。通过标准接口，SDK是完全可配置和可扩展的。包括签名算法、日志框架和状态存储，组件可以轻松进出SDK。SDK为事务处理、成员服务、节点遍历和事件处理提供了API。提供了多种风格的SDK：Node.js、Java和Python。

#### 状态数据库
当前状态数据被保存一个状态数据库中，以便链码可以方便地读和查询。支持的数据库包括levelDB和couchDB。

#### 系统链
包含一个在系统级别定义网络的配置区块。系统链存在于排序服务中，与通道类似，具有包含以下信息的初始配置：参与者组织的根证书和排序服务节点、策略、OSN(排序服务节点)监听地址以及配置详细信息。对整个网络的任何改变（例如一个新的组织加入或新增一个排序节点）将导致新的配置区块被添加到系统链中。  
系统链可以被认为是通道或通道组的通用绑定。例如，一系列金融机构可以组成一个联盟（通过系统链表示），然后再根据联盟或业务议程创建通道。  

#### 事务
为了排序、验证和提交而被递交的调用或实例化结果。调用是一个为了从账本读或写数据的请求。实例化是一个为了在通道中启动和实例化链码的请求。应用客户端收集从背书peer返回的调用或实例化响应，包装结果集和背书为一个事务，事务为了排序、验证和提交而被递交。

#### 排序者(Orderer)
构成排序服务的网络实体之一。一个排序服务节点（OSN）的集合，用来将事务排序进区块。在“独奏”的情况下，只需要一个OSN。事务被“广播”给排序者，然后作为区块“交付”到适当的通道。  

#### 背书者(Endorser)
一个特定的peer角色，背书peer负责模拟事务，并且防止不稳定或不确定的事务通过网络。事务以事务提议的形式发送给背书者。所有的背书peer同时也是committer peer（即他们写账本）。  

#### 提交者(Committer)  
一个特定的peer角色，负责将有效事务写入某通道的账本。一个peer可以同时充当背书者和提交者，当很多时候只是充当提交者。  


## 备忘
[Hyperledger Composer](https://hyperledger.org/projects/composer)  
[Hyperledger Composer教程](https://developer.ibm.com/courses/all/ibm-blockchain-foundation-developer/?course=begin#11982)，含composer的安装。  
[另一个官方文档](http://fabrictestdocs.readthedocs.io/en/latest/glossary.html)  
### 问题
#### VM翻墙问题
```
$ curl -sSL https://goo.gl/fMh2s3 | bash
```
如果执行后无反应，这是因为该链接被 墙。设置proxy:
```
$ export https_proxy=http://10.180.36.75:25378
$ export http_proxy=http://10.180.36.75:25378
```
25378是宿主机的翻&墙软件端口号，ip是宿主机ip。  
