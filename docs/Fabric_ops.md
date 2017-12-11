## 成员服务提供者(MSP)
本文提供了有关MSP的设置和最佳实践的详细信息。

成员服务提供商（MSP）是一个旨在提供成员维护体系结构抽象的组件。

特别是，MSP将发布和验证证书背后的所有密码学机制和协议以及用户认证抽象出来。MSP可以定义他们自己的身份概念，以及这些身份管理（身份验证）和认证（签名生成和验证）的规则。

Hyperledger Fabric区块链网络可以由一个或多个MSP管理。这提供了成员资格维护的模块化，以及跨不同成员标准和体系结构的互操作性。

在本文的其余部分中，我们将详细介绍由Hyperledger Fabric支持的MSP实现的设置，并讨论关于其使用的最佳实践。

### MSP配置
要创建一个MSP实例，需要在每个peer和orderer的本地指定其配置。

首先，对于每一个MSP名称需要以引用MSP在网络中指定（例如msp1，org2和org3.divA）。这个名称代表一个组织或组织单元(OU)，通道中会引用这个名称。这也被称为MSP标识符或MSP ID。MSP标识符在每个MSP实例中必须唯一的。
MSP的默认实现中，需要指定一组参数，用于身份(证书)验证和签名验证。这些参数在[RFC5280](http://www.ietf.org/rfc/rfc5280.txt)中描述。包括：
- 一些自签名(X.509)证书，构成了信任根(root of trust)  
- 一些X.509中间CA证书  
- 一些X.509证书，代表MSP的管理员，管理员证书由某个信任根签署  
- 一些这个MSP的OU证书  
- 一些撤销证书(CRL)  
- 一些自签名证书(X.509)，用于TLS
- 一些X.509证书，表示中间TLS CA证书

### 怎样生成MSP证书和密钥?
1. 使用[Openssl](https://www.openssl.org/)。需要强调的是Fabric不支持RSA密钥证书。  
2. 使用`cryptogen`工具，在[快速开始](http://hyperledger-fabric.readthedocs.io/en/latest/getting_started.html)中解释过。  
3. 使用[Hyperledger Fabric CA](http://hyperledger-fabric-ca.readthedocs.io/en/latest/)也可以生成配置MSP的密钥和证书。  

### 在peer&oderer端建立MSP
为peer或oderer建立一个本地MSP，管理员需要创建一个文件夹(如`$MY_PATH/mspconfig`)，下面包含6个子文件夹和一个文件：  
1. 一个`admincerts`文件夹，包含几个PEM文件，每个文件对应一个管理员证书  
2. 一个`cacerts`文件夹，包含几个PEM文件，每个文件对应到一个根CA证书
3. (可选)一个`intermediatecerts`文件夹，包含几个PEM文件，每个文件对应一个中间CA证书  
4. (可选)一个`config.yaml`文件，包含所考虑的OU的信息  
5. (可选)一个`crls`文件夹，包含了撤销证书列表(CRL)  
6. 一个`keystore`文件夹，包含一个PEM文件，是节点的签名密钥，不支持RSA密钥  
7. 一个`signcerts`文件夹，包含一个PEM文件，是节点的X.509证书  
8. (可选)一个`tlscacerts`文件夹，包含几个PEM文件，每个对应到一个TLS根CA证书  
9. (可选)一个`tlsintermediatecerts`文件夹，包含几个PEM文件，每个对应一个中间TLS CA证书   

在节点的配置文件中(对peer是core.yaml，对orderer是orderer.yaml)，需要指定到mspconfig文件夹的路径，和节点MSP的身份(Id)。到msconfig的路径是相对于FABRIC_CFG_PATH，对于peer由参数`mspConfigPath`的值定义，对于orderer由参数`LocalMSPDir`值定义。节点MSP的身份，对于peer由参数`localMspId`的值定义，对于orderer由参数`LocalMSPID`的值定义。  
这些变量可以被环境变量覆盖，对于peer节点由前缀为CORE的环境变量覆盖(如CORE_PEER_LOCALMSPID)，对于orderer节点由前缀为ORDERER的环境变量覆盖(如ORDERER_GENERAL_LOCALMSPID)。  
注意，对于建立orderer，需要生成和提供系统通道的创世区块(genesis block)到orderer节点。MSP配置对这个区块的需求在下一节讲到。  
重新配置一个“本地”MSP只能手工进行，需要peer和orderer进程重启。在后续版本中，我们的目标是提供在线/动态重新配置（即使用一个用节点管理系统链码来避免停止节点）。

### 通道MSP建立
在创建系统时，需要指定出现在网络中的所有MSP的验证参数，并且包括在系统通道的创世区块中。回想一下前文讲到的组成MSP身份的MSP验证参数，信任证书的根、中间CA和管理员证书，还有OU规范和CRL。系统创世区块被提供给orderer（在orderer的创建阶段），允许它们认证通道创建请求。如果区块包括两个相同身份的MSP，oderer会拒绝，使网络自举失败。  
对于应用通道，通道的创世区块包含了管理通道的MSP验证组件。我们强调这是应用的责任：确保在指示其一个或多个peer加入通道之前，通道的创世区块(或最新的配置区块)中包含了正确的MSP配置信息。  
在使用`configtxgen`工具启动一个通道时，需要配置通道MSP，办法是将MSP的验证参数包含在`mspconfig`文件夹，有在`configtx.yaml`的相应章节设置文件夹路径。  
重新配置通道的MSP，包括MSP的CA更新CRL公告，通过MSP的管理员证书之一的所有者创建`config_update`对象来实现。管理员管理的客户端应用会将这个更新广播到MSP出现的通道中。  

## 通道配置(configtx)
Hyperledger Fabric区块链网络的共享配置被保存在一个配置事务集合中，每个通道一个计划。每个配置事务通常叫做**configtx**。  
通道配置有如下重要属性：
1. 版本(**Versioned**)：配置的所有元素都有一个关联的版本，每次修改都会更新。此外，每次更新配置都会收到一个顺序号。  
2. 权限(**Permissioned**)：配置的每个元素有一个关联的策略，用于管理是否允许对该元素进行修改。任何拥有以前configtx副本（并且没有其他信息）的人都可以根据这些策略验证新配置的有效性。
3. 层次(**Hierarchical**)：根配置组包含子组，每个层次组都有关联的值和策略。这些政策可以利用层次结构从下层策略中推导出一个层次的策略。
### 解剖一个配置
配置被保存在区块的一个类型为`HeaderType_CONFIG`的事务中，该区块中没有其他事务。这些区块被称为**配置区块**，第一个区块被称为**创世区块**(Genesis Block)。  
配置的原型结构存储在`fabric/protos/common/configtx.proto`。

### 排序系统通道配置
排序系统通道需要定义排序参数，和创建通道的合伙人。对一个排序服务必须有一个排序系统通道，它是创建的第一个通道（指引导时）。推荐不要在排序系统通道的创世配置中定义应用段，但可以在测试时这样做。注意，对排序系统通道具有读权限的成员可以看到所有通道的创建，所以这个通道的访问权限需要严格控制。  

### 应用通道配置
通道的应用配置设计用于应用类型的事务。它的定义类似于：
```
&ConfigGroup{
    Groups: map<string, *ConfigGroup> {
        "Application":&ConfigGroup{
            Groups:map<String, *ConfigGroup> {
                {{org_name}}:&ConfigGroup{
                    Values:map<string, *ConfigValue>{
                        "MSP":msp.MSPConfig,
                        "AnchorPeers":peer.AnchorPeers,
                    },
                },
            },
        },
    },
}
```
### 通道创建
当排序节点收到一个不存在的通道的`CONFIG_UPDATE`，排序节点就假定这是个通道创建请求，然后执行下列操作：
1. 排序节点对通道创建请求的合伙人身份进行验证。它通过查看顶层group的`Consortium`值来验证。  
2. 
## 通道配置(configtxgen)
本文档介绍了`configtxgen`用于操作Hyperledger Fabric通道配置的实用程序的用法。

目前，该工具主要侧重于生成用于引导orderer的创世区块，但是将来要增强生成新的通道配置以及重新配置现有通道。  

### 引导orderer
```
$ configtxgen -profile <profile_name> -outputBlock orderer_genesisblock.pb
```
将在当面目录下创建一个`orderer_genesisblock.pb`文件。这个创世区块被用于引导排序系统通道，该通道被orderer用于授权和编排其他通道的创建。默认情况下，由`configtxgen`生成，编码进创世区块的通道ID是`testchainid`。  

为了利用生成的创世区块，在启动orderer之前，需要设定环境变量：
```
ORDERER_GENERAL_GENESISMETHOD=file
ORDERER_GENERAL_GENESISFILE=$PWD/orderer_genesisblock.pb
```
或者修改配置文件orderer.yaml，将上述值包含进文件中。  

### 创建一个通道
```
$ configtxgen -profile <profile_name> -channelID <channel_name> -outputCreateChannelTx <tx_filename>
```
这将输出一个`Envelope`消息并广播出去来创建一个通道。  
### 显示一个配置
除了能够生成配置，`configtxgen`工具还有查看配置的能力。  
它支持查看配置区块和配置事务。可以分别使用查看标志`-inspectBlock`和`-inspectChannelCreateTx`并在后面附加文件路径来输出一个JSON串来显示配置信息。  
还可以对查看标志进行组合，例如：
```
$ configtxgen -channelID foo -outputBlock foo_genesisblock.pb -inspectBlock foo_genesisblock.pb
2017-11-02 17:56:04.489 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
2017-11-02 17:56:04.564 EDT [common/tools/configtxgen] doOutputBlock -> INFO 002 Generating genesis block
2017-11-02 17:56:04.564 EDT [common/tools/configtxgen] doOutputBlock -> INFO 003 Writing genesis block
2017-11-02 17:56:04.564 EDT [common/tools/configtxgen] doInspectBlock -> INFO 004 Inspecting block
2017-11-02 17:56:04.564 EDT [common/tools/configtxgen] doInspectBlock -> INFO 005 Parsing genesis block
(json略)
```
上述命令先生成区块，再显示它。  

## 用configtxlator重新配置
### 概述
创建`configtxlator`工具的目的是支持不依赖SDK的重新配置。通道配置被当作一个事务存储在通道的配置区块中，可以被直接操作，如在bdd行为测试中。不过，在撰写本文时，SDK本身不支持直接操作配置，因此该configtxlator工具旨在提供API，供任何SDK的使用者与之交互以协助配置更新。

工具名称是configtx和translator的混合，旨在表达该工具只是在两者之间进行转换。它不会生成配置。它不递交或检索配置。它不会自己修改配置，它只是在configtx格式的不同视图之间提供一些双向操作。

标准用法为：

1. SDK检索最新的配置  
2. `configtxlator` 产生人类可读的配置版本  
3. 用户或应用程序编辑配置  
4. `configtxlator`用于计算配置的变更  
5. SDK递交签名和递交配置  

`configtxlator`工具暴露了一个真正的无状态的REST API来与配置元素进行交互。这些REST组件支持将原生配置格式转换为人类可读的JSON格式，或者根据两种配置之间的差异来计算配置变更。

由于`configtxlator`服务故意不包含任何密钥材料或其他秘密信息，因此不包含任何授权或访问控制。预期的典型部署将在本地与应用程序一起作为沙盒容器来运行，因而`configtxlator`是个为消费者提供的专用进程。

### 运行configtxlator
`configtxlator`工具可以与其他Hyperledger Fabric平台相关的二进制文件一起下载。 详细信息查看download-platform-specific-binaries。

该工具可能被配置为侦听不同的端口，您也可以使用`--port`和`--hostname`标志来指定主机名。要查看完整的命令和标志，请运行`configtxlator --help`。

该二进制包将启动一个监听指定端口的http服务器，然后就可以处理请求了。

要启动`configtxlator`服务器：
```
$ configtxlator start
2017-06-21 18:16:58.248 HKT [configtxlator] startServer -> INFO 001 Serving HTTP requests on 0.0.0.0:7059
```
### Proto翻译
为了扩展性，并且由于某些字段必须被签名，许多proto字段被存储为字节。这使得无法使用`jsonpb`包来完成原生proto到JSON的翻译。作为代替，`configtxlator`暴露了一个REST组件来做更复杂的翻译。

为了将proto转换为人类可读的JSON等价物，只需将二进制proto发布到REST地址`http://$SERVER:$PORT/protolator/decode/<message.Name>`，其中`<message.Name>`是消息的完全限定的proto名称。

例如，要解码一个配置区块并另存为`configuration_block.pb`，请运行以下命令：:
```
$ curl -X POST --data-binary @configuration_block.pb http://127.0.0.1:7059/protolator/decode/common.Block
```
为了转换人类可读JSON版本的prot消息，只需要把JSON消息发送到`http://$SERVER:$PORT/protolator/encode/<message.Name`，这里`<message.Name>`仍然是消息的完全限定的proto名称。  
例如，重编码区块另存为`configuration_block.json`，运行命令：
```
$ curl -X POST --data-binary @configuration_block.json http://127.0.0.1:7059/protolator/encode/common.Block
```
任何配置相关proto，包括`common.Block`、`common.Envelope`、`common.ConfigEnvelope`、`common.ConfigUpdateEnvelope`
、`common.Config`和`common.ConfigUpdate`都是这些URL有效目标。未来，可能会增加其它proto编码类型，如背书者事务。  

### 配置更新计算
给定两种不同的配置，可以计算在它们之间转换的配置更新。只需将两个`common.Config`proto编码的配置作为`multipart/formdata`、original作为字段`original`和updated作为字段`updated`，发送到`http://$SERVER:$PORT/configtxlator/compute/update-from-configs`。

例如，给定原始配置文件`original_config.pb`和更新的配置文件`updated_config.pb`，对于通道`desiredchannel`：
```
$ curl -X POST -F channel=desiredchannel -F original=@original_config.pb -F updated=@updated_config.pb http://127.0.0.1:7059/configtxlator/compute/update-from-configs
```
### 自举例子
启动`configtxlator`:
```
$ configtxlator start
2017-05-31 12:57:22.499 EDT [configtxlator] main -> INFO 001 Serving HTTP requests on port: 7059
```
首先，为排序系统通道生成一个创世区块：
```
$ configtxgen -outputBlock genesis_block.pb
2017-05-31 14:15:16.634 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
2017-05-31 14:15:16.646 EDT [common/configtx/tool] doOutputBlock -> INFO 002 Generating genesis block
2017-05-31 14:15:16.646 EDT [common/configtx/tool] doOutputBlock -> INFO 003 Writing genesis block
```
将创世区块解码为人类可编辑格式：
```
$ curl -X POST --data-binary @genesis_block.pb http://127.0.0.1:7059/protolator/decode/common.Block > genesis_block.json
```
编辑生成的文件`genesis_block.json`，或者用程序操作它。这里我们使用JOSN CLI工具`jq`。为了简单，我们编辑通道的批大小，因为它是个简单的数字。然而，所有都可以编辑，包括策略和MSP。

首先，让我们建立一个环境变量来保存指向json中属性的路径：
```
$ export MAXBATCHSIZEPATH=".data.data[0].payload.data.config.channel_group.groups.Orderer.values.BatchSize.value.max_message_count"
```
然后，让我们显示这个属性的值：
```
$ jq "$MAXBATCHSIZEPATH" genesis_block.json
10
```
现在，我们设置新的批大小，并显示这个新值：
```
$ jq “$MAXBATCHSIZEPATH = 20” genesis_block.json > updated_genesis_block.json
$ jq “$MAXBATCHSIZEPATH” updated_genesis_block.json
20
```
创世区块现在准备好被重编码为用于自举的原生proto格式：
```
$ curl -X POST --data-binary @updated_genesis_block.json http://127.0.0.1:7059/protolator/encode/common.Block > updated_genesis_block.pb
```
`updated_genesis_block.pb`文件现在可以作为创世区块用于一个排序系统通道的自举。

### 重新配置的例子
利用另外的终端窗口，使用默认配置启动排序服务，临时自举器会创建一个`testchainid`排序系统通道。
```
ORDERER_GENERAL_LOGLEVEL=debug orderer
```
重新配置一个通道的操作非常类似于改变一个创世配置。

首先，取得配置区块proto：
```
$ peer channel fetch config config_block.pb -o 127.0.0.1:7050 -c testchainid
2017-05-31 15:11:37.617 EDT [msp] getMspConfig -> INFO 001 intermediate certs folder not found at [/home/yellickj/go/src/github.com/hyperledger/fabric/sampleconfig/msp/intermediatecerts]. Skipping.: [stat /home/yellickj/go/src/github.com/hyperledger/fabric/sampleconfig/msp/intermediatecerts: no such file or directory]
2017-05-31 15:11:37.617 EDT [msp] getMspConfig -> INFO 002 crls folder not found at [/home/yellickj/go/src/github.com/hyperledger/fabric/sampleconfig/msp/intermediatecerts]. Skipping.: [stat /home/yellickj/go/src/github.com/hyperledger/fabric/sampleconfig/msp/crls: no such file or directory]
Received block:  1
Received block:  1
2017-05-31 15:11:37.635 EDT [main] main -> INFO 003 Exiting.....
```
然后，发送配置区块到`configtxlator`服务进行解码：
```
curl -X POST --data-binary @config_block.pb http://127.0.0.1:7059/protolator/decode/common.Block > config_block.json
```
从区块中提取配置节：
```
$ jq .data.data[0].payload.data.config config_block.json > config.json
```
编辑配置，把它另存为一个新的`updated_config.json`。这里，我们设置批大小为30。
```
$ jq ".channel_group.groups.Orderer.values.BatchSize.value.max_message_count = 30" config.json  > updated_config.json
```
对原始配置和变更的配置进行重编码到proto格式：
```
$ curl -X POST --data-binary @config.json http://127.0.0.1:7059/protolator/encode/common.Config > config.pb
$ curl -X POST --data-binary @updated_config.json http://127.0.0.1:7059/protolator/encode/common.Config > updated_config.pb
```
现在，两个配置都进行适当编码，发送它们到*configtxlator*服务，去计算它们之间的配置变更。
```
$ curl -X POST -F original=@config.pb -F updated=@updated_config.pb http://127.0.0.1:7059/configtxlator/compute/update-from-configs -F channel=testchainid > config_update.pb
```
这时，计算出来的配置变更已经准备好了。传统上，使用一个SDK签名和包装这个消息。然而，为了仅使用peer cli，*configtxlator*也可以用于完成这个任务。

首先，我们对配置变更(ConfigUPdate)进行解码，以便可以用文本方式操作它：
```
$ curl -X POST --data-binary @config_update.pb http://127.0.0.1:7059/protolator/decode/common.ConfigUpdate > config_update.json
```
然后，我们把它包裹进一个信封(envelope)消息：
```
$ echo '{"payload":{"header":{"channel_header":{"channel_id":"testchainid", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' > config_update_as_envelope.json
```
接下来，将其转换回完整的配置事务的proto格式：
```
$ curl -X POST --data-binary @config_update_as_envelope.json http://127.0.0.1:7059/protolator/encode/common.Envelope > config_update_as_envelope.pb
```
最后，发送配置变更事务到排序服务去执行一个配置变更。
```
$ peer channel update -f config_update_as_envelope.pb -c testchainid -o 127.0.0.1:7050
```
### 增加一个组织
先启动`configtxlator`：
```
$ configtxlator start
2017-05-31 12:57:22.499 EDT [configtxlator] main -> INFO 001 Serving HTTP requests on port: 7059
```
使用`SampleDevModeSolo`profile选项启动排序服务。
```
$ ORDERER_GENERAL_LOGLEVEL=debug ORDERER_GENERAL_GENESISPROFILE=SampleDevModeSolo orderer
```
增加一个组织的过程与批大小的例子非常类似。然而，不同与设置批大小，一个新组织定义在应用级别。添加一个组织稍微牵扯一点，因为我们必须先创建一个通道，然后修改它的成员组成。  

## 背书策略
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html)  
背书策略是用来指导peer如何决定事务是否得到适当的赞同。当peer收到一个事务时，作为事务验证流程的一部分，它调用与事务的链码相关联的VSCC(验证系统链码)来确定事务的有效性。回忆一下，一个事务包含来自任何背书peer的一个或多个背书。VSCC的任务是做出以下决定：
- 所有的背书是有效的（也就是通过预期的消息它们来自有效证书的有效签名）  
- 有恰当数量的签名  
- 背书来自预期的源  

背书策略是指定第二和第三点的一种方式。  

### CLI中的背书策略语法
(本节中的身份都是principal)
在CLI中，策略用一种简单的语言来表达，即基于身份的布尔表达式。  
一个身份通过MSP描述，它的任务是验证签名者身份和签名者角色存在于MSP中。当前，支持两种角色，**member**(成员)和**admin**(管理员)。身份用`MSP`.`ROLE`描述，这里`MSP`是MSP ID(必填)，`ROLE`是`member`或`admin`二选一。验证身份(principal)的例子是`Org0.admin`(`Org0`MSP的任何管理员)或`Org1.member`(`Org1`MSP的任何成员)。  
这个语言的语法是：  
```
EXPR(E[, E...])
```
这里`EXPR`是`AND`或`OR`，表示两个布尔表达式。`E`可以是一个身份或另一个嵌入的`EXPR`。（运算符在前面，这被称为波兰语表示法）   

例如：  
- `AND('Org1.member', 'Org2.member', 'Org3.member')` 要求三位身份分别签名  
- `OR('Org1.member', 'Org2.member')` 两身份中任何一个签名都可以  
- `OR('Org1.member', AND('Org2.member', 'Org3.member'))` 要求`Org1` MSP成员的一个签名，或`Org2`MSP成员和`Org3`MSP成员的分别签名  
### 定义链码的背书策略
使用这个语言，链码部署者可以为某个链码指定特定策略。注意，默认策略是需要`DEFAULT`MSP成员的一个签名。用CLI实例化链码时，没有指定策略就使用默认策略。  
实例化时用`-P`选项指定策略。例如：
```
$ peer chaincode instantiate -C <channelid> -n mycc -P "AND('Org1.member', 'Org2.member')"
```
这个命令部署链码`mycc`，使用的策略是`AND('Org1.member', 'Org2.member')`，即同时需要Org1和Org2的成员签署事务。  

# 架构
## 架构解释
Hyperledger Fabric架构具有以下优点：  
- **Chaincode信任灵活性**。该架构将链码（区块链应用）信任假设与排序信任假定分开。换句话说，排序服务可以由一组节点（orderer）提供，并容忍的一些节点的失效或欺诈；并且对于每个链码，背书者可能不同。  
- **可扩展性**。由于背书节点只负责特定链码，与排序节点无关，系统可以比通过相同节点完成这些功能更好地扩展。具体而言，当不同的链码指定无关的背书节点时，这引入了背书节点之间的分区机制从而允许链码的并行运行（背书）。  
- **保密**。该架构便于部署对于其事务的内容和状态更新具有机密性要求的链码。
- **共识模块化**。该体系结构是模块化的，并允许插件式的共识（即排序服务）实现。  
### 1.系统架构
区块链是由多个彼此通信的节点组成的分布式系统。区块链运行程序称为链码，保存状态和总帐数据，并执行事务。链码是链码调用事务操作的核心元素。事务必须被“背书”，只有背书的事务可能会被提交，并对状态产生影响。管理功能和参数存在一个或多个特殊的链码，统称为系统链码。  
#### 1.1 事务
事务可能有两种类型：
- **部署事务**创建新的链码并将一个程序作为参数。当部署事务成功执行时，链码就被安装在区块链上。  
- **调用(Invoke)事务**在先前部署的链码的上下文中执行操作。调用事务是指链码及其提供的某个函数。链码会执行指定的函数 - 这可能涉及修改相应的状态，并返回一个输出。  
如后面所述，部署事务是调用事务的特殊情况，其中创建新链的部署事务相当于系统链码上的调用事务。  
**备注**： 这个文件目前假设一个事务要么创建新的链码，要么调用已经部署的链码提供的操作。本文档尚未描述：a）对查询（只读）事务（包含在v1中）的优化，b）对交叉链码事务（post-v1特性）的支持。  
#### 1.2 区块链数据结构
**1.2.1状态**  
区块链的最新状态（或简称为状态）被建模为版本化的键值库（KVS），其中键名称和值是任意的blob。这些条目由运行在区块链上的链码（应用程序）通过`put`和`get` KVS来操纵。状态被持久化存储，状态的更新被日志记录。请注意，采用版本化的KVS作为状态模型，实现可以使用实际的KVS，还可以使用RDBMS或任何其他解决方案。  
**1.2.2账本**  
账本提供了一个在系统运行过程中发生的所有成功状态变化（我们称为有效事务）和不成功的改变状态尝试（我们称为无效事务）的可验证历史。  
账本是由排序服务（参见1.3.3节）构建，是一个由事务区块（有效或无效）组成的有序哈希链。哈希链定义了账本中的总的区块顺序，每个区块都包含一个完全有序的事务数组。这会在所有事务中强加顺序。  
账本保存在所有peer节点，并可保存在部分排序节点(可选)。在排序节点上下文中，我们称账本为`OrdererLedger`，然而在peer上下文中，我们称账本为`PeerLedger`。`PeerLedger`与`OrdererLedger`的区别在于，peer在本地维护一个位掩码(bitmask)，将有效的事务从无效的事务中分离出来（更多细节见第XX章）。  
像第XX节（v1之后的功能）中所述的那样，peer可能会删除`PeerLedger`。排序节点维护`OrdererLedger`是为了容错性和`PeerLedger`的可用性，并且可以决定在什么时候修剪(prune)它，前提是排序服务的属性（参见第1.3.3节）得到维护。  

账本允许peer重放所有事务的历史记录并重建状态。因此，如1.2.1节所述的状态是可选的数据结构。  
#### 1.3节点
节点是区块链的通信实体。一个“节点”只是一个逻辑功能，也就是不同类型的多个节点可以在同一个物理服务器上运行。重要的是节点如何分组在“信任域”中，并与控制它们的逻辑实体相关联。  
存在三种类型的节点：  
1. **客户端**或**提交客户端**：它发出实际调用事务到背书节点，广播提议事务到排序服务。  
2. **Peer**：它提交事务、维护状态和一个账本副本(参看1.2节)。此外，peer还可以具有背书者角色。  
3. **排序服务节点**或**orderer**：运行通信服务的节点，实现交付担保，如原子或全部排序广播。  

**1.3.1客户端** 
客户代表代表最终用户行事的实体。它必须连接到peer与区块链进行通信。客户端可以连接到其选择的任何peer端。客户端创建并调用事务。  

如第2节详细描述的那样，客户端同时与peer和排序服务通信。  

**1.3.2 Peer**  
peer以区块的形式从排序服务接收顺序的状态更新，并维护状态和账本。  

peer还可以担当**背书peer**的特殊角色(或称**endorser**)。背书peer的特殊功能是针对特定的链码进行的，并且包含在提交前的背书事务中。每个链码可以指定一个背书策略，策略可以指向一组背书peer。该策略为有效的事务背书定义了必要和充分的条件（通常是一组背书签名），如后面的第2和第3节所述。有一种部署新链码的特殊部署事务，它的（部署）背书策略是指定为系统链码的背书政策。  

**1.3.3排序服务节点(Orderer)**  
排序服务(orderer)是一个通信架构，它提供投递担保。排序服务可以用不同的方式实现：从中心式服务（例如在开发和测试中使用）到针对不同网络和节点失效模型的分布式协议。

排序服务为客户和peer提供共享的*通信通道*，为包含事务的消息提供广播服务。客户端连接到该通道，并可以在该通道上广播消息，然后传送给所有peer。该通道支持所有消息的原子交付，也就是全排序交付和（特定实现）可靠性的消息通信。换句话说，通道向所有连接的peer输出相同的消息，并以相同的逻辑顺序输出到所有peer。这种原子通信保证也被称为*全序广播*、*原子广播*或分布式系统下的*共识*。通信消息是包含在区块链状态中的候选事务。  

**分区(排序服务通道)**。排序服务可以支持多通道，类似发布/订阅（pub / sub）消息系统的主题。客户端可以连接到给定的通道，然后可以发送消息并获取到达的消息。通道可以被认为是分区 - 连接到一个通道的客户端不知道其他通道的存在，但客户端可能连接到多个通道。尽管Hyperledger Fabric中包含的一些排序服务实现支持多个通道，但为了简化表示，在本文档的其余部分，我们假设排序服务包含单个通道/主题。  

**排序服务API**。peer通过排序服务提供的接口连接到排序服务提供的通道。排序服务API包含两个基本操作（很常见的*异步事件*）：  
- `broadcast(blob)`: 客户端调用这个广播二进制消息`blob`到通道。这在BFT上下文中也叫做`request(blob)`，当发送请求到一个服务时。  
- `deliver(seqno, prevhash, blob)`：(略)

**账本和区块信息**。账本(参看1.2.2节)包含排序服务输出的所有数据。简而言之，它是一系列`deliver(seqno, prevhash, blob)`事件，根据前面所述的`prevhash`计算形成一个哈希链。  
大多数情况下，出于效率原因，不输出单个事务（blob），排序服务将对blob进行分组(批处理)，并在一个`deliver`事件输出区块。在这种情况下，排序服务必须强制和传达每个块内的blob的确定性排序。区块中的blob数量可以由排序服务实现动态地选择。  

下面为便于表述，我们定义排序服务属性（本小节的其余部分），并解释事务背书的工作流程。假定一个blob产生一个`deliver`事件。一个区块对应一组顺序的`deliver`事件（每个blob一个事件）。区块本身也对应一个`deliver`事件，依靠排序服务，多个区块顺序排列组成区块链。  

### 2.事务背书的基本流程
#### 2.1 客户端创建一个事务并将它发送到选择的背书peers  
为了调用一个事务，客户端发送一个`PROPOSE`消息给它所选择的一组背书peer（可能不是同一时间 - 见2.1.2节和2.3节）。对于给定的`chaincodeID`客户端根据背书策略(看第3节)可以获得一组背书peer。例如，根据给定`chiancodeID`事务可以发送给所有背书peer。也就是说，一些背书人可能会离线，其他人可能会反对并选择不赞成事务。发起客户端利用目前可用的背书节点尝试满足策略表达式的要求。  

接下来，我们首先详细描述PROPOSE消息格式，然后讨论提交客户端和背书人之间可能的交互模式。  
**2.1.1 PROPOSE消息格式**  
**2.1.2 消息模式**  

#### 2.2 背书peer模拟一个事务并生成一个背书签名  
从客户端收到`<PROPOSE,tx,[anchor]>`消息后，背书peer`epID`首先验证客户端签名`clientSig`，然后模拟一个事务。如果客户端指定了`anchor`，则背书peer模拟事务的方法是，在本地KVS中读取与版本号`anchor`相匹配的keys。  
模拟一个事务包括背书peer临时性执行一个事务(`txPayload`)（调用事务中`chaincodeID`指定的链码）和背书peer本地保存的状态副本。（用状态副本和临时事务可以得到一个临时的新状态）  
作为执行的结果，背书peer计算*读版本依赖*(`readset`)和*状态更新*(`writeset`)，在数据库语言中也叫*MVCC+postimage info*。  
回想一下状态由键值对组成。所有的键值对条目是版本化的，也就是每个条目都包含有序的版本信息，当通过一个key更新库中的值时版本号会增长。peer解释(模拟执行)事务，记录链码访问的键值对，但不会真的更新状态。进一步来说：
- 在背书peer执行事务前，给定状态`s`，对于事务读取的每个key`k`，键值对`(k,s(k).version)`被添加到`readset`。  
- 此外，对于每个key`k`事务更新为新值`v'`，键值对`(k,v')`被添加到`writeset`。或者，`v'`可能是以前值(`s(k).value`)的新值的增量。  

如果客户端在`PROPOSE`消息中指定了`anchor`，则客户端指定的`anchor`必须等于背书peer模拟事务时产生的`readset`。  
然后，peer将内部`tran-proposal`（即`tx`）转发作为其背书事务逻辑的组成部分，称为**背书逻辑**。默认情况下，peer中的背书逻辑接受`tran-proposal`并简单地签名`tran-proposal`。然而，背书逻辑可能解释任意函数，例如，用`tran-proposal`和`tx`作为输入与遗留系统交互来判断是否批准事务。  
如果背书逻辑决定对一个事务进行背书，它发送`<TRANSACTION-ENDORSED, tid, tran-proposal,epSig>`消息到发起客户端(`tx.clientID`)，这里：
- `tran-proposal := (epID,tid,chaincodeID,txContentBlob,readset,writeset)`，`txContentBlob`是链码/事务指定信息。  
- `epSig`是背书peer在`tran-proposal`上的签名  

另外，如果背书逻辑拒绝对事务签名，背书界面会发送一个`(TRANSACTION-INVALID, tid, REJECTED)`消息到发起客户端。  
注意，背书节点在这一步不会修改状态，因事务模拟而生成的更新不会影响状态。  

#### 2.3 发起客户端收集事务背书并广播到排序服务
发起客户端等待，直到它收到“足够的”消息和签名(`TRANSACTION-ENDORSED, tid, *, *`)，得出事务提议被背书的结论。正如2.1.2节所讨论的，这可能涉及一个或多个与背书人交互的往返。  

“足够的”的确切数量取决于链码背书策略（另见第3节）。如果背书策略得到满足，事务就获得背书; 注意它还没有提交。来自背书peer的签名`TRANSACTION-ENDORSED`消息集合将建立一个背书的事务，称为*背书*(`endorsement`)。

如果发起客户端没有收集到事务的背书，则放弃该事务，并选择稍后重试。  

对于含有有效背书的事务，我们现在开始使用排序服务。发起客户端使用`broadcast(blob)`调用排序服务，这里`blob=endorsement`。如果客户端没有直接调用排序服务的能力，可以通过自己选择的peer代理它的广播。这样的peer必须被客户信任：不会从`endorsement`删除任何消息，除非事务被认为是无效的。但是请注意，代理peer可能不会编造有效的endorsement。  

#### 2.4 排序服务
当一个事件`deliver(seqno, prevhash, blob)`发生，并且peer已经应用了blob的序列号低于`seqno`的所有的状态更新，peer做下面这些：
- 它根据`blob.tran-proposal.chaincodeID`指向的链码策略检查`blob.endorsement`看是否有效。  
- 在一个典型的情况下，它也验证依赖关系（blob.endorsement.tran-proposal.readset）同时没有被违反。在更复杂的用例中，背书的`tran-proposal`字段可能有所不同，在这种情况下，背书策略（第3部分）指定了状态如何演变。  
（省略一些）

### 3. 背书策略
#### 3.1 背书策略规范
一个**策略**，是对事务进行背书的条件。区块链peer拥有一套预先设定的背书策略，由安装特定链码的`deploy`事务引入。背书策略可以参数化，这些参数可以由`deploy`事务指定。  

为了保证区块链和安全属性，这套认可策略应该是一组经过验证的策略，确保有限的执行时间（终止），确定性，性能和安全保证。  

不允许动态增加背书策略，日后可以予以支持。  

#### 3.2 对背书策略的事务评估
只有通过策略背书，事务才被宣布有效。链码的调用事务首先必须获得链码策略的背书，否则将不会被提交。这是通过发起客户端和peer之间的交互来进行的，如第2节所述。  

形式上，背书策略是对背书的一个判断，并可能进一步陈述评估为“真”或“假”。对于部署事务，根据系统范围的策略（例如从系统链码）获得背书。

(省略一些)

# Fabric实战

## MSP配置(cryptogen)
要运行cryptogen，需要一个指定一个配置文件`crypto-config.yaml`，如：
```
$ cd /opt/fabric-samples/first-network
$ cryptogen generate --config=./crypto-config.yaml
```
执行后，会在当前目录下创建一个`crypto-config`文件夹。`crypto-config`文件夹的结构如下：
```
crypto-config
    ordererOrganizations
        example.com
            ca
            msp
            orderers
            tlsca
            users
    peerOrganizations
        org1.example.com
            ca
            msp
            orderers
            tlsca
            users
        org2.example.com
            (同上)
```
上述目录结构是根据`crypto-config.yaml`的内容生成的。`crypto-config.yaml`的内容如下：
```yaml
OrdererOrgs:
  - Name: Orderer
    Domain: example.com
    Specs:
      - Hostname: orderer
PeerOrgs:
  - Name: Org1
    Domain: org1.example.com
  - Name: Org2
    Domain: org2.example.com
```
上述`crypto-config.yaml`是fabric-sample带的示范配置文件，它将Orderer和Peer两种节点的MSP都定义到了同一个文件中。而在实际的生产环境下，Orderer节点和Peer节点一般部署在不同的虚拟机中，则两种节点的`crypto-config.yaml`配置文件将有所不同。  

要搭建一个Fabric网络，首先应建立Orderer节点。而Orderer节点存在着一个系统区块链，上面保存了整个Fabric网络的基础信息。要创建一个区块链，先用`cryptogen`工具生成MSP密码文件，再用`configtxgen`工具生成创世区块。更具体的，应先生成Orderer节点的创世区块。  

## 生成创世区块(configtxgen)
`configtxgen`运行时需要查找配置文件`configtx.yaml`，这个配置文件的位置是靠环境变量来指定的：
```
$ export FABRIC_CFG_PATH=/opt/fabric-samples/first-network
$ configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
```
执行后生成了一个“创世区块”文件`genesis.block`。如果用vi打开这个文件看看，发现里面有9个证书，应该是3个属于orderer，6个属于peer。  
`TwoOrgsOrdererGenesis`来自哪里？这就需要开发配置文件`configtx.yaml`来看看了：
```yaml
Profiles:
    TwoOrgsOrdererGenesis:
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1
                    - *Org2
    TwoOrgsChannel:
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
                - *Org2
Organizations:
    - &OrdererOrg
        Name: OrdererOrg
        ID: OrdererMSP
        MSPDir: crypto-config/ordererOrganizations/example.com/msp

    - &Org1
        Name: Org1MSP
        ID: Org1MSP
        MSPDir: crypto-config/peerOrganizations/org1.example.com/msp

        AnchorPeers:
            - Host: peer0.org1.example.com
              Port: 7051

    - &Org2
        Name: Org2MSP
        ID: Org2MSP
        MSPDir: crypto-config/peerOrganizations/org2.example.com/msp

        AnchorPeers:
            - Host: peer0.org2.example.com
              Port: 7051
(略)
```
除了`profile`的定义`TwoOrgsOrdererGenesis`外，配置文件中另一个与`configtxgen`的执行有关的属性是一个目录：
```
MSPDir: crypto-config/ordererOrganizations/example.com/msp
```
这个目录中密码文件由`cryptogen`生成。如果之前没有用`cryptogen`生成这些文件，`configtxgen`执行时会提示这些上述目录不存在。

### 用configtxgen查看区块信息
```
$ export FABRIC_CFG_PATH=/opt/bcnet
$ configtxgen -profile TwoOrgsOrdererGenesis -inspectBlock ./channel-artifacts/genesis.block
```
`configtxgen`会到FABRIC_CFG_PATH环境变量定义的目录下找`configtx.yaml`这个配置文件，并按配置文件中的定义先找`profile`，根据`profile`的值所对应的`MSPDir`下寻找密钥文件，然后利用密钥文件对创世区块进行解密并转换为json格式输出。

有的创世区块可以不加`-profile`参数就能正常显示，原因不明。  

## 生成通道(configtxgen)
```
$ export FABRIC_CFG_PATH=/opt/fabric-samples/first-network
$ configtxgen -profile TwoOrgChannel -outputCreateChannelTx ./channel-artifacts/channel1.tx -channelID channel1
```

## peer加入通道(peer)
peer直接在宿主机上直接运行会报错，一般进入`cli`容器内执行:
```
$ docker exec -it cli bash
$$ peer --help
```

