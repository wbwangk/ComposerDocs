# 将Hyperledger Composer区块链业务网络部署到Hyperledger Fabric（多个组织）

本教程将演示多个组织方案中的管理员必须采取的步骤将区块链业务网络部署到Hyperledger Fabric实例，包括如何生成Hyperledger Composer配置。

建议您首先阅读前面的教程，演示如何将区块链业务网络部署到单个组织的Hyperledger Fabric实例，因为它将更详细地解释一些概念。

本教程将介绍如何部署区块链业务网络向Hyperledger Fabric网络跨越两个组织，`Org1`和`Org2`。本教程将以不同类型的步骤进行介绍，具体取决于哪个组织应遵循该步骤。

第一步是两个组织遵循：

## (灰)示例步骤：遵循Org1和Org2的步骤

组织`Org1`由Alice代表，绿色：

## (绿)示例步骤：遵循Org1的步骤

组织`Org2`由Bob代表，紫色：

## (紫)示例步骤：Org2遵循的步骤

您可以自行执行这些步骤，或与朋友或同事配合，一起按照步骤操作。

让我们开始吧！

## 先决条件

如果您安装了开发环境，则需要先停止开发环境提供的Hyperledger Fabric：

```bash
cd ~/fabric-tools
./stopFabric.sh
./teardownFabric.sh
```

克隆下面的GitHub仓库：

```bash
git clone -b issue-6978 https://github.com/sstone1/fabric-samples.git
```

按照[构建您的第一个网络教程](http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html)，确保您使用上一步中克隆的GitHub存储库。您不能克隆和使用GitHub存储库的Hyperledger Fabric版本，因为它当前缺少本教程所需的更改。**重要事项**确保您使用的网址中有`latest`，以确保您使用Hyperledger Fabric1.1的预览

## (灰)第一步：启动Hyperledger Fabric网络

为了遵循本教程，您必须启动Hyperledger Fabric网络。

本教程将假设您使用“Hyperledger Fabric [构建您的第一个网络”教程](http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html)中提供的Hyperledger Fabric网络。我们将把这个Hyperledger Fabric网络称为BYFN（建立你的第一个网络）网络。

您现在可以启动BYFN网络。您必须指定在“构建您的第一个网络”教程中未指定的其他标志。这是因为我们希望将CouchDB用作世界状态数据库，并且我们希望为每个组织启动证书颁发机构（CA）。

```
./byfn.sh -m generate
./byfn.sh -m up -s couchdb -a
```

如果命令成功，BYFN网络启动，您将看到以下输出：
```
========= All GOOD, BYFN execution completed ===========


_____   _   _   ____
| ____| | \ | | |  _ \
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/
```

接下来，删除您的钱包中可能存在的任何业务网络卡片。忽略任何表示无法找到业务网络卡片的错误是安全的：

```
composer card delete -n PeerAdmin@byfn-network-org1
composer card delete -n PeerAdmin@byfn-network-org2
composer card delete -n alice@tutorial-network
composer card delete -n bob@tutorial-network
composer card delete -n admin@tutorial-network
composer card delete -n PeerAdmin@fabric-network
```

然而，任何其他类型的故障都可能表明您的卡存储中有来自旧版本的Hyperledger Composer的卡，您将不得不删除文件系统卡存储，如下所示

```
rm -fr ~/.composer
```

## (灰)第二步：探索Hyperledger Fabric网络

这一步将探索BFYN网络配置和组件。需要配置详细信息才能完成后续步骤。

#### 组织

该BYFN网络由两个组织：`Org1`和`Org2`。组织`Org1`使用域名`org1.example.com`。 `Org1`的成员服务提供商（MSP）称为`Org1MSP`。组织`Org2`使用域名`org2.example.com`。`Org2`的MSP称为 `Org2MSP`。在本教程中，您将部署一个可以与组织`Org1`和`Org2`交互的区块链业务网络。

#### 网络组件

Hyperledger Fabric网络由多个组件组成：

- `Org1`的两个peer节点，名为`peer0.org1.example.com` 和`peer1.org1.example.com`。  
  请求端口为`peer0`7051。  
  事件集线器端口为`peer0`7053。  
  请求端口为`peer1`8051。  
  事件集线器端口为`peer1`8053。  

- 一个`Org1`的CA（证书颁发机构），名为`ca.org1.example.com`。  
  CA端口是7054。  

- 'Org2`的两个peer节点` peer0.org2.example.com`和` peer1.org2.example.com`。   
  请求端口为`peer0`9051。   
  事件中心端口为`peer0`9053。   
  请求端口为`peer1`10051。  
  事件中心端口为`peer1`10053。  

- 一个`Org2`的CA（证书颁发机构），名为`ca.org2.example.com`。  
  CA端口是8054。  

- 单个排序节点，名为`orderer.example.com`。  
  排序端口是7050。  

这些组件在Docker容器中运行。在Docker容器中运行Hyperledger Composer时，上面的名称（例如，`peer0.org1.example.com`）可用于与Hyperledger Fabric网络进行交互。

本教程将在Docker主机上运行Hyperledger Composer命令，而不是从Docker网络内运行。这意味着Hyperledger Composer命令必须使用`localhost`主机名和暴露的容器端口与Hyperledger Fabric网络进行交互。

所有网络组件都使用TLS来加密通信。您将需要所有网络组件的证书颁发机构（CA）证书才能连接到这些网络组件。CA证书可以在包含byfn.sh脚本的目录中找到。

排序节点的CA证书：
```
crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt
```

`Org1`的CA证书：
```
crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

`Org2`的CA证书：
```
crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
```

稍后您将使用这些文件与Hyperledger Fabric网络进行交互。

#### 用户

组织`Org1`配置了一个名为的用户`Admin@org1.example.com`。此用户是管理员。

用户`Admin@org1.example.com`有一组存储在目录中的证书和私钥文件：
```
crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
```

组织`Org2`配置了一个名为的用户`Admin@org2.example.com`。此用户是管理员。

用户`Admin@org2.example.com`有一组存储在目录中的证书和私钥文件：
```
crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
```

稍后您将使用其中一些文件与Hyperledger Fabric网络进行交互。

除了管理员，`Org1`和`Org2`的CA（证书颁发机构）的已经配置了默认用户。此默认用户的登记ID为 `admin`和登记密码为`adminpw`。但是，此用户无权部署区块链业务网络。

#### 通道

已经创建了一个名为`mychannel`的通道。这四个peer节点- ，`peer0.org1.example.com`，`peer1.org1.example.com`，`peer0.org2.example.com`和`peer1.org2.example.com`已经加入到这个通道。

#### 连接配置文件

我们需要一个基础连接配置文件来描述这个结构网络，然后可以将它们给`alice`和`bob`，用来为其组织定制。
```json
{
    "name": "hlfv1",
    "x-type": "hlfv1",
    "version": "1.0.0",
    "channels": {
        "mychannel": {
            "orderers": [
                "orderer.example.com"
            ],
            "peers": {
                "peer0.org1.example.com": {
                    "endorsingPeer": true,
                    "chaincodeQuery": true,
                    "eventSource": true
                },
                "peer1.org1.example.com": {
                    "endorsingPeer": true,
                    "chaincodeQuery": true,
                    "eventSource": true
                },
                "peer0.org2.example.com": {
                    "endorsingPeer": true,
                    "chaincodeQuery": true,
                    "eventSource": true
                },
                "peer0.org2.example.com": {
                    "endorsingPeer": true,
                    "chaincodeQuery": true,
                    "eventSource": true
                }
            }
        }
    },
    "organizations": {
        "Org1": {
            "mspid": "Org1MSP",
            "peers": [
                "peer0.org1.example.com",
                "peer1.org1.example.com"
            ],
            "certificateAuthorities": [
                "ca.org1.example.com"
            ]
        },
        "Org2": {
            "mspid": "Org2MSP",
            "peers": [
                "peer0.org2.example.com",
                "peer1.org2.example.com"
            ],
            "certificateAuthorities": [
                "ca.org2.example.com"
            ]
        }
    },
    "orderers": {
        "orderer.example.com": {
            "url": "grpcs://localhost:7050",
            "grpcOptions": {
                "ssl-target-name-override": "orderer.example.com"
            },
            "tlsCACerts": {
                "pem": "INSERT_ORDERER_CA_CERT"
            }
        }
    },
    "peers": {
        "peer0.org1.example.com": {
            "url": "grpcs://localhost:7051",
            "eventUrl": "grpcs://localhost:7053",
            "grpcOptions": {
                "ssl-target-name-override": "peer0.org1.example.com"
            },
            "tlsCACerts": {
                "pem": "INSERT_ORG1_CA_CERT"
            }
        },
        "peer1.org1.example.com": {
            "url": "grpcs://localhost:8051",
            "eventUrl": "grpcs://localhost:8053",
            "grpcOptions": {
                "ssl-target-name-override": "peer1.org1.example.com"
            },
            "tlsCACerts": {
                "pem": "INSERT_ORG1_CA_CERT"
            }
        },
        "peer0.org2.example.com": {
            "url": "grpcs://localhost:9051",
            "eventUrl": "grpcs://localhost:9053",
            "grpcOptions": {
                "ssl-target-name-override": "peer0.org2.example.com"
            },
            "tlsCACerts": {
                "pem": "INSERT_ORG2_CA_CERT"
            }
        },
        "peer0.org1.example.com": {
            "url": "grpcs://localhost:10051",
            "eventUrl": "grpcs://localhost:10053",
            "grpcOptions": {
                "ssl-target-name-override": "peer1.org2.example.com"
            },
            "tlsCACerts": {
                "pem": "INSERT_ORG2_CA_CERT"
            }
        }
    },
    "certificateAuthorities": {
        "ca.org1.example.com": {
            "url": "https://localhost:7054",
            "caName": "ca-org1"
        },
        "ca.org2.example.com": {
            "url": "https://localhost:8054",
            "caName": "ca-org2"
        }
    }
}

```

在使用证书时，建议创建一个临时工作目录来创建Composer连接配置文件：
```
mkdir -p /tmp/composer/org1
mkdir -p /tmp/composer/org2
```

您需要将文本的所有`INSERT_ORG1_CA_CERT`实例替换为peer节点的CA证书，以便`Org1`：使用以下命令将pem文件转换为可以嵌入上述连接配置文件中的文件。
```
awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt > /tmp/composer/org1/ca-org1.txt
```

复制文件`/tmp/ca-org1.txt`的内容并替换文本`INSERT_ORG1_CA_CERT`。现在应该看起来像这样
```
"pem": "-----BEGIN CERTIFICATE-----\nMIICNTCCAdygAwIBAgIRAMNvmQpnXi7uM19BLdha3MwwCgYIKoZIzj0EAwIwbDEL\nMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG\ncmFuY2lzY28xFDASBgNVBAoTC2V4YW1wbGUuY29tMRowGAYDVQQDExF0bHNjYS5l\neGFtcGxlLmNvbTAeFw0xNzA2MjYxMjQ5MjZaFw0yNzA2MjQxMjQ5MjZaMGwxCzAJ\nBgNVBAYTAlVTMRMwEQYDVQQIEwpDYWxpZm9ybmlhMRYwFAYDVQQHEw1TYW4gRnJh\nbmNpc2NvMRQwEgYDVQQKEwtleGFtcGxlLmNvbTEaMBgGA1UEAxMRdGxzY2EuZXhh\nbXBsZS5jb20wWTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAASJn3QUVcKCp+s6lSPE\nP5KlWmE9rEG0kpECsAfW28vZQSIg2Ez+Tp1alA9SYN/5BtL1N6lUUoVhG3lz8uvi\n8zhro18wXTAOBgNVHQ8BAf8EBAMCAaYwDwYDVR0lBAgwBgYEVR0lADAPBgNVHRMB\nAf8EBTADAQH/MCkGA1UdDgQiBCB7ULYTq3+BQqnzwae1RsnwQgJv/HQ5+je2xcDr\nka4MHTAKBggqhkjOPQQDAgNHADBEAiB2hLiS8B1g4J5Qbxu15dVWAZTAXX9xPAvm\n4l25e1oS+gIgBiU/aBwSxY0uambwMB6xtQz0ZE/D4lyTZZcW9SODlOE=\n-----END CERTIFICATE-----\n"
```

您需要将文本的所有实例替换`INSERT_ORG2_CA_CERT`为peer节点的CA证书，以便`Org2`：使用以下命令将pem文件转换为可以嵌入上述连接配置文件中的文件。
```
awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt > /tmp/composer/org2/ca-org2.txt
```

复制文件`/tmp/ca-org2.txt`的内容并替换文本`INSERT_ORG2_CA_CERT`。

您需要将文本的所有实例替换`INSERT_ORDERER_CA_CERT`为排序节点的CA证书：使用以下命令将pem文件转换为可以嵌入到上述连接配置文件中的文件。
```
awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt > /tmp/composer/ca-orderer.txt
```

复制文件`/tmp/ca-orderer.txt`的内容并替换文本`INSERT_ORDERER_CA_CERT`。

完成后，将该文件另存为`connection-network.json`。

运行以下命令为Org1和Org2创建连接配置文件：
```
cd /tmp/composer
cp connection-network.json ./org1/connection-org1.json
cp connection-network.json ./org2/connection-org2.json
```

此连接配置文件现在描述了Fabric网络设置，所有peer、排序节点和证书颁发机构都是网络的一部分，它定义了参与网络的所有组织，还定义了此网络上的通道。Hyperledger Composer只能与单个通道交互，因此只能定义一个通道。

## (绿)第三步：自定义Org1的连接配置文件

这是一个为`alice`所属组织配置的例子，配置在client部分并使用了定制的超时，因此将以下内容添加到上述连接配置文件`connection-network.json`中的`version`属性之后和`channel`属性之前，并将其保存为`connection-org1.json`。
```json
    "client": {
        "organization": "Org1",
        "connection": {
            "timeout": {
                "peer": {
                    "endorser": "300",
                    "eventHub": "300",
                    "eventReg": "300"
                },
                "orderer": "300"
            }
        }
    },
```

所以配置文件的这部分应该看起来像
```json
    ...
    "version": "1.0.0",
    "client": {
        "organization": "Org1",
        "connection": {
            "timeout": {
                "peer": {
                    "endorser": "300",
                    "eventHub": "300",
                    "eventReg": "300"
                },
                "orderer": "300"
            }
        }
    },
    "channel": {
    ...
```

## (紫)第四步：构建Org2的连接配置文件

为`bob`重复相同的过程，但这次指定组织为`Org2`并保存文件`connection-org2.json`，它应该有一个类似的部分
```json
    ...
    "version": "1.0.0",
    "client": {
        "organization": "Org2",
        "connection": {
            "timeout": {
                "peer": {
                    "endorser": "300",
                    "eventHub": "300",
                    "eventReg": "300"
                },
                "orderer": "300"
            }
        }
    },
    "channel": {
    ...
```

## (绿)第五步：找到Org1的Hyperledger Fabric管理员的证书和私钥

我们的Hyperledger Fabric网络的管理员是一名用户`Admin@org1.example.com`。该用户的证书和私钥文件存储在以下目录中：
```
crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
```

您必须首先找到该用户的证书文件。证书是身份的公共部分。证书文件可以在`signcerts`子目录中找到，名称是`Admin@org1.example.com-cert.pem`。

接下来，您必须找到该用户的私钥文件。私钥用于以这种身份对交易进行签名。私钥文件可以在`keystore`子目录中找到。私钥文件的名称是一个很长的以`_sk`为后缀的十六进制字符串，例如`78f2139bfcfc0edc7ada0801650ed785a11cfcdef3f9c36f3c8ca2ebfa00a59c_sk`。每次生成配置名称都会更改。

记住这两个文件的路径，或将它们复制到与`connection-org1.json`在第三步中创建的连接配置文件相同的目录中。您将在接下来的步骤中需要这些文件。

## (紫)第六步：找到Org2的Hyperledger Fabric管理员的证书和私钥

我们的Hyperledger Fabric网络的管理员是一名用户`Admin@org2.example.com`。该用户的证书和私钥文件存储在以下目录中：
```
crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
```

您必须首先找到该用户的证书文件。证书是身份的公共部分。证书文件可以在`signcerts`子目录中找到，名称是`Admin@org2.example.com-cert.pem`。

接下来，您必须找到该用户的私钥文件。私钥用于以这种身份对交易进行签名。私钥文件可以在`keystore`子目录中找到。私钥文件的名称是一个很长的的`_sk`为后缀的十六进制字符串，例如`d4889cb2a32e167bf7aeced872a214673ee5976b63a94a6a4e61c135ca2f2dbb_sk`。每次生成配置名称都会更改。

请记住这两个文件的路径，或将它们复制到与`connection-org2.json`您在步骤4中创建的连接配置文件相同的目录中。您将在接下来的步骤中需要这些文件。

## (绿)第七步：为Org1的Hyperledger Fabric管理员创建业务网络卡片

在这一步中，您将创建业务网络卡片供管理员用来将区块链业务网络部署到Hyperledger Fabric网络。

运行`composer card create`命令以使用`Org1`的连接置文件创建业务网络卡片。您必须指定所有三个文件的路径，您可以在之前的步骤中创建或找到它们（注意：*sk*文件将有所不同）。
```
composer card create -p connection-org1.json -u PeerAdmin -c Admin@org1.example.com-cert.pem -k 78f2139bfcfc0edc7ada0801650ed785a11cfcdef3f9c36f3c8ca2ebfa00a59c_sk -r PeerAdmin -r ChannelAdmin
```

如果该命令成功，则将一个叫`PeerAdmin@byfn-network-org1.card`的业务网络卡片文件写入当前目录。

## (紫)第八步：为Org2的Hyperledger Fabric管理员创建业务网络卡片

在这一步中，您将创建业务网络卡片供管理员用来将区块链业务网络部署到Hyperledger Fabric网络。

运行`composer card create`命令以使用连接配置文件为`Org2`创建业务网络卡片。您必须指定您在以前的步骤中创建或找到的所有三个文件的路径：
```
composer card create -p connection-org2.json -u PeerAdmin -c Admin@org2.example.com-cert.pem -k d4889cb2a32e167bf7aeced872a214673ee5976b63a94a6a4e61c135ca2f2dbb_sk -r PeerAdmin -r ChannelAdmin
```

如果该命令成功，则将一个叫`PeerAdmin@byfn-network-org2.card`的业务网络卡片文件写入当前目录。

## (绿)第九步：为Org1的Hyperledger Fabric管理员导入商业网卡

运行`composer card import`命令将`Org1`的业务网络卡片导入钱包：
```
composer card import -f PeerAdmin@byfn-network-org1.card --name PeerAdmin@byfn-network-org1
```

如果命令成功，一个名为`PeerAdmin@byfn-network-org1`的业务网络卡片将被导入钱包。

##(紫) 第十步：导入Org2的Hyperledger Fabric管理员的业务网络卡片

运行`composer card import`命令将`Org2`的业务网络卡片导入钱包：
```
composer card import -f PeerAdmin@byfn-network-org2.card --name PeerAdmin@byfn-network-org2
```

如果命令成功，一个名为`PeerAdmin@byfn-network-org2`的业务网络卡片将被导入钱包。

## (绿)步骤十一：将Hyperledger Composer运行时安装到Org1的Hyperledger Fabric peer节点上

运行以下`composer runtime install`命令将Hyperledger Composer运行时安装到您在步骤三中创建的连接配置文件中指定的`Org1`的所有Hyperledger Fabric peer节点上：
```
composer runtime install -c PeerAdmin@byfn-network-org1 -n tutorial-network
```

## (紫)第十二步：将Hyperledger Composer运行时安装到Org2的Hyperledger Fabric peer节点上

运行以下`composer runtime install`命令，将Hyperledger Composer运行时安装到您在步骤4中创建的连接配置文件中指定的`Org2`的所有Hyperledger Fabric peer节点上：
```
composer runtime install -c PeerAdmin@byfn-network-org2 -n tutorial-network
```

## (灰)第十三步：确定业务网络的背书策略

正在运行的业务网络有一个背书策略，该策略定义了组织必须支持的交易的规则，然后才能将其交付给区块链。默认情况下，业务网络部署的背书政策规定只有一个组织必须先签署交易，然后才能将其交付给区块链。

在现实世界的区块链业务网络中，多个组织希望交易在提交到区块链前先经过他们背书，所以默认的背书策略是不合适的。相反，您可以在启动业务网络时指定自定义的背书策略。

你可以在Hyperledger Fabric文档[背书策略](https://hyperledger-fabric.readthedocs.io/en/release/endorsement-policies.html)中找到背书策略的详细信息。

> 请注意，用于业务网络的背书策略必须采用Hyperledger Fabric Node.js SDK使用的JSON格式。这与Hyperledger Fabric CLI使用的简单背书策略格式有所不同，您可以在Hyperledger Fabric文档中看到这种格式。

使用以下内容创建一个叫`endorsement-policy.json`的背书策略文件，并将其保存到磁盘。你将在后面的步骤中使用这个文件，所以记住你的位置！
```json
{
    "identities": [
        {
            "role": {
                "name": "member",
                "mspId": "Org1MSP"
            }
        },
        {
            "role": {
                "name": "member",
                "mspId": "Org2MSP"
            }
        }
    ],
    "policy": {
        "2-of": [
            {
                "signed-by": 0
            },
            {
                "signed-by": 1
            }
        ]
    }
}
```

您刚刚创建的背书策略指出，在提交到区块链之前，`Org1`和`Org2`必须先对交易背书。如果`Org1`或`Org2`不对交易背书，或对交易结果不同意，交易将被业务网络拒绝。

## (灰)步骤十四：了解并选择业务网络管理员

业务网络启动时，业务网络必须配置一组初始参与者。这些参与者将负责引导业务网络，并将其他参与者引入业务网络。在Hyperledger Composer中，我们将这些初始参与者称为业务网络管理员。

在我们的业务网络中，组织`Org1`和`Org2`拥有平等的权利。每个组织将为业务网络提供一个业务网络管理员，这些业务网络管理员将加载其组织中的其他参与者。`Org1`的业务网络管理员将是Alice，而`Org2`的业务网络管理员将是Bob。

在启动业务网络时，必须将所有业务网络管理员的证书（身份的公共部分）传递给执行命令的组织以启动业务网络。业务网络启动后，所有业务网络管理员都可以使用自己的身份与业务网络进行交互。

您可以在[部署业务网络中](https://hyperledger.github.io/composer/next/business-network/bnd-deploy.html)找到有关业务网络管理员的更多信息。

## (绿)第十五步：获取Org1的业务网络管理员证书

运行`composer identity request`命令以获取Alice的业务网络管理员的证书，用于`Org1`：
```
composer identity request -c PeerAdmin@byfn-network-org1 -u admin -s adminpw -d alice
```

命令选项`-u admin`的`-s adminpw`要与Hyperledger Fabric CA（证书颁发机构）注册的默认用户相对应。

证书将被放置在当前工作目录中叫`alice`的目录中。有三个证书文件创建，但只有两个是重要的。它们是`admin-pub.pem`证书（包括公钥）和`admin-priv.pem`私钥。只有`admin-pub.pem`文件适合与其他组织共享。`admin-priv.pem`文件必须保密，因为它可以代表签发机构用于签署交易。

## (紫)第十六步：检索Org2的业务网络管理员证书

运行`composer identity request`命令以获取Bob的业务网络管理员证书，用于`Org2`：
```
composer identity request -c PeerAdmin@byfn-network-org2 -u admin -s adminpw -d bob
```

命令选项`-u admin`的`-s adminpw`要与Hyperledger Fabric CA（证书颁发机构）注册的默认用户相对应。

certficates将被放置在当前工作目录叫`bob`的目录中。有三个证书文件创建，但只有两个是重要的。它们是`admin-pub.pem`证书（包括公钥）和`admin-priv.pem`私钥。只有`admin-pub.pem`文件适合与其他组织共享。`admin-priv.pem`文件必须保密，因为它可以代表签发机构用于签署交易。

## (绿)第十七步：启动业务网络

运行`composer network start`命令启动业务网络。只`Org1`需要执行这个操作。此命令使用`endorsement-policy.json`第十三步创建的`admin-pub.pem`文件以及Alice和Bob在步骤十五和十六中创建的文件，因此您必须确保所有这些文件都可以通过此命令访问：
```
composer network start -c PeerAdmin@byfn-network-org1 -a tutorial-network@0.0.1.bna -o endorsementPolicyFile=endorsement-policy.json -A alice -C alice/admin-pub.pem -A bob -C bob/admin-pub.pem
```

一旦这个命令完成，业务网络将被启动。Alice和Bob都可以访问业务网络，开始建立业务网络，并从其各自的组织中加载其他参与者。但是，Alice和Bob都必须使用他们在前面的步骤中创建的证书创建新的业务网络卡片，以便他们可以访问业务网络。

## (绿)步骤十八：创建一个业务网络卡片作为Org1访问业务网络

运行`composer card create`命令创建 `Org1`的业务网络管理员Alice用来访问业务网络的业务网络卡片：
```
composer card create -p connection-org1.json -u alice -n tutorial-network -c alice/admin-pub.pem -k alice/admin-priv.pem
```

运行`composer card import`命令导入刚创建的业务网络卡片：
```
composer card import -f alice@tutorial-network.card
```

运行`composer network ping`命令测试与区块链业务网络的连接：
```
composer network ping -c alice@tutorial-network
```

如果命令成功完成，则应在命令的输出中看到完全限定的参与者标识符`org.hyperledger.composer.system.NetworkAdmin#alice`。现在，您可以使用此业务网络卡片与区块链业务网络进行交互，并与组织中的其他参与者进行交互。

## (紫)第十九步：创建一个商业网卡作为Org2访问业务网络

运行`composer card create`命令创建`Org2`的业务网络管理员Bob 用来访问业务网络的业务网络卡片：
```
composer card create -p connection-org2.json -u bob -n tutorial-network -c bob/admin-pub.pem -k bob/admin-priv.pem
```

运行`composer card import`命令导入刚创建的业务网络卡片：
```
composer card import -f bob@tutorial-network.card
```

运行`composer network ping`命令测试与区块链业务网络的连接：
```
composer network ping -c bob@tutorial-network
```

如果命令成功完成，则应在命令的输出中看到完全限定的参与者标识符`org.hyperledger.composer.system.NetworkAdmin#bob`。现在，您可以使用此业务网络卡片与区块链业务网络进行交互，并与组织中的其他参与者进行交互。

## (灰)结论

在本教程中，您已经看到了如何配置Hyperledger Composer所需的所有信息，以连接到跨多个组织的Hyperledger Fabric网络，以及如何部署跨Hyperledger Fabric网络中所有组织的区块链业务网络。
