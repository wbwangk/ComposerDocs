## 部署Hyperledger Composer网络到多组织Fabric
[原文：Deploying a Hyperledger Composer blockchain business network to Hyperledger Fabric (multiple organizations)](https://hyperledger.github.io/composer/unstable/tutorials/deploy-to-fabric-multi-org.html)   

这个教程示范了在多组织场景下的管理员部署一个区块链业务网络到某Hyperledger Fabric实例必须执行的几个步骤，包括怎样生成Hyperledger Composer配置。

建议首先完成前面的教程，也就是示范怎样为单个组织将一个区块链业务网络部署到某Hyperledger Fabric实例，因为它更详细地解释了一些概念。

这个教程将涵盖怎样部署一个区块链业务网络到某个跨越两个组织(`Org1`和`Org2`)的Hyperledger Fabric网络。这个教程会根据不同组织执行的步骤展现为不同的方式。

首先是两个组织都要执行的步骤显示的样子：  
**◎示例步骤：一个Org1和Org2都执行的步骤**  

表示组织`Org1`执行的步骤的样子:  
**①示例步骤：一个Org1执行的步骤**  

表示组织`Org2`执行的步骤的样子：  
**②示例步骤：一个Org2执行的步骤**  

你可以自己执行这些步骤，或者与朋友或同事一起执行这些步骤。  
让我们开始！  

### ◎准备
如果你已经安装了开发环境，你需要首先停止开发环境提供的Hyperledger Fabric：
```bash
cd ~/fabric-tools
./stopFabric.sh
./teardownFabric.sh
```

克隆下面的Github库：
```bash
git clone -b issue-6978 https://github.com/sstone1/fabric-samples.git
```
按照[建立你的首个网络](https://hyperledgercn.github.io/hyperledgerDocs/build_network_zh/)教程，确保你使用了上面步骤克隆的Github库。你不能克隆和使用Hyperledger Fabric版本的Github库，因为它当前没有更新到这个教程需要的内容。

### ◎步骤一：启动一个Hyperledger Fabric网络
为了遵照这个教程，你必须启动一个Hyperledger Fabric网络。

这个教程假定你使用了Hyperledger Fabric的[建立你的首个网络](https://hyperledgercn.github.io/hyperledgerDocs/build_network_zh/)教程中提供的 Hyperledger Fabric网络。我会称这个 Hyperledger Fabric网络为BYFN(Building Your First Network)网络。

你现在可以启动BYFN网络。你必须使用额外标志，这个标志在BYFN教程中并没有使用。这是因为我们想使用CouchDB作为世界状态数据库，并且我们想为每个组织启动一个CA(Certificate Authority)。
```bash
./byfn.sh -m generate
./byfn.sh -m up -s couchdb -a
```
(估计作者用的BYFN版本旧，现在的1.1.0版BYFN默认启动couchdb，但不启动CA容器)  

如果你的命令执行成功，BYFN网络被启动，你会看到下面的输出：
```
========= All GOOD, BYFN execution completed ===========

_____   _   _   ____   
| ____| | \ | | |  _ \  
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/
```
然后，删除存在于你钱包中的所有业务网络卡片。这是为了避免业务网络卡片没有找到的错误状态：
```bash
composer card delete -n PeerAdmin@byfn-network-org1-only
composer card delete -n PeerAdmin@byfn-network-org1
composer card delete -n PeerAdmin@byfn-network-org2-only
composer card delete -n PeerAdmin@byfn-network-org2
composer card delete -n alice@tutorial-network
composer card delete -n bob@tutorial-network
composer card delete -n admin@tutorial-network
composer card delete -n PeerAdmin@fabric-network
```
### ◎步骤二：探索Hyperledger Fabric网络
这一步骤会探索BYFN网络配置和组件。为了完成后续步骤需要一些配置明细。

#### 组织
BYFN网络由两个组织组成：`Org1`和`Org2`。组织`Org1`使用域名`org1.example.com`。`Org1`的成员服务提供者(MSP)叫`Org1MSP`。组织`Org2`使用域名`org2.example.com`。`Org2`的MSP叫`Org2MSP`。在这个教程中，你会部署一个区块链业务网络，在里面组织`Org1`和`Org2`互相交互。

#### 网络组件
Hyperledger Fabric网络由下面几个组件组成：
 - Org1的两个peer节点，叫peer0.org1.example.com 和 peer1.org1.example.com  
   - peer0的请求端口是7051  
   - peer0的事件hub端口是7053  
   - peer1的请求端口是8051  
   - peer1的事件hub端口是8053  
 - 一个Org1的CA(Certificate Authority)，叫 ca.org1.example.com  
   - CA端口是7054  
 - Org2的两个peer节点，叫peer0.org2.example.com 和 peer1.org2.example.com  
   - peer0的请求端口是9051  
   - peer0的事件hub端口是9053  
   - peer1的请求端口是10051  
   - peer1的事件hub端口是10053  
 - 一个Org1的CA(Certificate Authority)，叫 ca.org1.example.com  
   - CA端口是7054  
 - 一个排序节点，叫orderer.example.com  
   - 排序端口是7050  

这些组件运行在Docker容器中。当在一个Docker容器中运行Hyperledger Composer，与Hyperledger Fabric网络交互可以使用上面的名字(如`peer0.org1.example.com`)。

这个教程会在Docker宿主机中运行Hyperledger Composer命令，而不是在Docker网络中。这意味着Hyperledger Composer命令与Hyperledger Fabric网络交互使用`localhost`作为主机名和容器暴露的端口。

所有这些网络组件使用TLS加密通信来确保安全。为了连接到这些网络组件，你会使用它们的CA(Certificate Authority)证书。CA证书所在目录可以在byfn.sh脚本中找到。

排序(orderer)节点的CA证书：
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
之后你会使用这些文件与Hyperledger Fabric网络交互。

#### 用户
组织`Org1`配置了一个叫`Admin@org1.example.com`的用户。这个用户是一个管理员。

用户`Admin@org1.example.com`有一组证书和私钥文件保存在这个目录：
```
crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
```
组织`Org2`配置了一个叫`Admin@org2.example.com`的用户。这个用户是一个管理员。

用户`Admin@org2.example.com`有一组证书和私钥文件保存在这个目录：
```
crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
```
之后你会使用这些文件与Hyperledger Fabric网络交互。

除了管理员，`Org1`和`Org2`的CA(Certificate Authority)还配置了一个默认用户。这个默认用户有一个叫`admin`的登记ID和为`adminpw`的登记密码。然而，这个用户没有部署区块链业务网络的权限。

#### 信道
创建了一个叫`mychannel`的信道。所有四个节点`peer0.org1.example.com`、`peer1.org1.example.com`、`peer0.org2.example.com`和`peer1.org2.example.com`已经加入了这个信道。

### ①步骤三：创建Org1的连接配置文件
`Org1`需要两个连接配置文件。一个连接配置文件仅包含属于`Org1`的peer节点，另一个连接配置文件包含属于`Org1`和`Org2`的peer节点。

创建一个叫`connection-org1-only.json`的连接配置文件，包含下列内容并保存到磁盘。这个连接配置文件仅包含属于`Org1`的peer节点。你会在后续步骤中使用这个文件，所以记住存放位置！
```json
{
    "name": "byfn-network-org1-only",
    "type": "hlfv1",
    "mspID": "Org1MSP",
    "peers": [
        {
            "requestURL": "grpcs://localhost:7051",
            "eventURL": "grpcs://localhost:7053",
            "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer0.org1.example.com"
        },
        {
            "requestURL": "grpcs://localhost:8051",
            "eventURL": "grpcs://localhost:8053",
            "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer1.org1.example.com"
        }
    ],
    "ca": {
        "url": "https://localhost:7054",
        "name": "ca-org1",
        "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
        "hostnameOverride": "ca.org1.example.com"
    },
    "orderers": [
        {
            "url" : "grpcs://localhost:7050",
            "cert": "INSERT_ORDERER_CA_CERT_FILE_PATH",
            "hostnameOverride": "orderer.example.com"
        }
    ],
    "channel": "mychannel",
    "timeout": 300
}
```
用指向`Org1`peer节点的CA证书文件的全路径名覆盖所有的`INSERT_ORG1_CA_CERT_FILE_PATH`文本：
```
crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```
用指向排序节点的CA证书文件的全路径名覆盖所有的`INSERT_ORDERER_CA_CERT_FILE_PATH`文本：
```
crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt
```

创建一个叫`connection-org1.json`的连接配置文件，包含下列内容并保存到磁盘。这个连接配置文件d包含属于`Org1`和`Org2`的peer节点。你会在后续步骤中使用这个文件，所以记住存放位置！
```json
{
    "name": "byfn-network-org1",
    "type": "hlfv1",
    "mspID": "Org1MSP",
    "peers": [
        {
            "requestURL": "grpcs://localhost:7051",
            "eventURL": "grpcs://localhost:7053",
            "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer0.org1.example.com"
        },
        {
            "requestURL": "grpcs://localhost:8051",
            "eventURL": "grpcs://localhost:8053",
            "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer1.org1.example.com"
        },
        {
            "requestURL": "grpcs://localhost:9051",
            "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer0.org2.example.com"
        },
        {
            "requestURL": "grpcs://localhost:10051",
            "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer1.org2.example.com"
        }
    ],
    "ca": {
        "url": "https://localhost:7054",
        "name": "ca-org1",
        "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
        "hostnameOverride": "ca.org1.example.com"
    },
    "orderers": [
        {
            "url" : "grpcs://localhost:7050",
            "cert": "INSERT_ORDERER_CA_CERT_FILE_PATH",
            "hostnameOverride": "orderer.example.com"
        }
    ],
    "channel": "mychannel",
    "timeout": 300
}
```
用指向`Org1`peer节点的CA证书文件的全路径名覆盖所有的`INSERT_ORG1_CA_CERT_FILE_PATH`文本：
```
crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```
用指向`Org2`peer节点的CA证书文件的全路径名覆盖所有的`INSERT_ORG2_CA_CERT_FILE_PATH`文本：
```
crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
```
用指向排序节点的CA证书文件的全路径名覆盖所有的`INSERT_ORDERER_CA_CERT_FILE_PATH`文本：
```
crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt
```
注意，这个连接配置文件包含了`Org2`peer节点的明细，它仅包含了请求端口，而没有包含事件hub端口。这是因为一个组织不能访问另一个组织的事件hub端口。

### ②步骤四：创建Org2的连接配置文件
`Org2`需要两个连接配置文件。一个连接配置文件仅包含属于`Org2`的peer节点，另一个连接配置文件包含属于`Org2`和`Org1`的peer节点。

创建一个叫`connection-org2-only.json`的连接配置文件，包含下列内容并保存到磁盘。这个连接配置文件仅包含属于`Org2`的peer节点。你会在后续步骤中使用这个文件，所以记住存放位置！
```json
{
    "name": "byfn-network-org2-only",
    "type": "hlfv1",
    "mspID": "Org2MSP",
    "peers": [
        {
            "requestURL": "grpcs://localhost:9051",
            "eventURL": "grpcs://localhost:9053",
            "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer0.org2.example.com"
        },
        {
            "requestURL": "grpcs://localhost:10051",
            "eventURL": "grpcs://localhost:10053",
            "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer1.org2.example.com"
        }
    ],
    "ca": {
        "url": "https://localhost:8054",
        "name": "ca-org2",
        "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
        "hostnameOverride": "ca.org2.example.com"
    },
    "orderers": [
        {
            "url" : "grpcs://localhost:7050",
            "cert": "INSERT_ORDERER_CA_CERT_FILE_PATH",
            "hostnameOverride": "orderer.example.com"
        }
    ],
    "channel": "mychannel",
    "timeout": 300
}
```
用指向`Org2`peer节点的CA证书文件的全路径名覆盖所有的`INSERT_ORG2_CA_CERT_FILE_PATH`文本：
```
crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
```
用指向排序节点的CA证书文件的全路径名覆盖所有的`INSERT_ORDERER_CA_CERT_FILE_PATH`文本：
```
crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt
```
创建一个叫`connection-org2.json`的连接配置文件，包含下列内容并保存到磁盘。这个连接配置文件包含属于`Org2`和`Org1`的peer节点。你会在后续步骤中使用这个文件，所以记住存放位置！
```json
{
    "name": "byfn-network-org2",
    "type": "hlfv1",
    "mspID": "Org2MSP",
    "peers": [
        {
            "requestURL": "grpcs://localhost:9051",
            "eventURL": "grpcs://localhost:9053",
            "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer0.org2.example.com"
        },
        {
            "requestURL": "grpcs://localhost:10051",
            "eventURL": "grpcs://localhost:10053",
            "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer1.org2.example.com"
        },
        {
            "requestURL": "grpcs://localhost:7051",
            "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer0.org1.example.com"
        },
        {
            "requestURL": "grpcs://localhost:8051",
            "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer1.org1.example.com"
        }
    ],
    "ca": {
        "url": "https://localhost:8054",
        "name": "ca-org2",
        "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
        "hostnameOverride": "ca.org2.example.com"
    },
    "orderers": [
        {
            "url" : "grpcs://localhost:7050",
            "cert": "INSERT_ORDERER_CA_CERT_FILE_PATH",
            "hostnameOverride": "orderer.example.com"
        }
    ],
    "channel": "mychannel",
    "timeout": 300
}
```
用指向`Org2`peer节点的CA证书文件的全路径名覆盖所有的`INSERT_ORG2_CA_CERT_FILE_PATH`文本：
```
crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
```
用指向`Org1`peer节点的CA证书文件的全路径名覆盖所有的`INSERT_ORG1_CA_CERT_FILE_PATH`文本：
```
crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```
用指向排序节点的CA证书文件的全路径名覆盖所有的`INSERT_ORDERER_CA_CERT_FILE_PATH`文本：
```
crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt
```
注意，这个连接配置文件包含了`Org1`peer节点的明细，它仅包含了请求端口，而没有包含事件hub端口。这是因为一个组织不能访问另一个组织的事件hub端口。

### ①步骤五：定位Org1的Hyperledger Fabric管理员的证书和私钥
我们Hyperledger Fabric网络的管理员是一个叫`Admin@org1.example.com`的用户。这个用户的证书和私钥存放在这个目录：
```
crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
```
你必须先为这个用户定位这个证书文件。证书是身份的公开部分。证书文件可以在子目录`signcerts`中找到，文件名是`Admin@org1.example.com-cert.pem`。

然后，你为这个用户定位私钥文件。私钥用于以这个身份签署交易。私钥文件可以在子目录`keystore`中找到。私钥文件名是一个长十六进制字符串，后缀是`_sk`，例如`78f2139bfcfc0edc7ada0801650ed785a11cfcdef3f9c36f3c8ca2ebfa00a59c_sk`。每次配置生成时这个名字会改变。

记住这些文件的路径，或者将它们复制到步骤三中创建的连接pfile文件`connection-org1.json`所在的目录。

### ②步骤六：定位Org2的Hyperledger Fabric管理员的证书和私钥

我们Hyperledger Fabric网络的管理员是一个叫`Admin@org2.example.com`的用户。这个用户的证书和私钥存放在这个目录：
```
crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
```
你必须先为这个用户定位这个证书文件。证书是身份的公开部分。证书文件可以在子目录`signcerts`中找到，文件名是`Admin@org2.example.com-cert.pem`。

然后，你为这个用户定位私钥文件。私钥用于以这个身份签署交易。私钥文件可以在子目录`keystore`中找到。私钥文件名是一个长十六进制字符串，后缀是`_sk`，例如`d4889cb2a32e167bf7aeced872a214673ee5976b63a94a6a4e61c135ca2f2dbb_sk`。每次配置生成时这个名字会改变。

记住这些文件的路径，或者将它们复制到步骤四中创建的连接pfile文件`connection-org2.json`所在的目录。

### ①步骤七：为Org1的Hyperledger Fabric管理员生成业务网络卡片

在这个步骤中你会为管理员创建业务网络卡片，用于将区块链业务网络部署到 Hyperledger Fabric网络。

运行`composer card create`命令创建一个业务网络卡片，使用包含`Org1`peer的连接配置文件。你必须指定三个文件的路径，无论新建还是前面步骤定位的(注意：*sk*文件不同)。
```bash
composer card create -p connection-org1-only.json -u PeerAdmin -c Admin@org1.example.com-cert.pem -k 78f2139bfcfc0edc7ada0801650ed785a11cfcdef3f9c36f3c8ca2ebfa00a59c_sk -r PeerAdmin -r ChannelAdmin
```
如果命令执行成功，一个叫`PeerAdmin@byfn-network-org1-only.card`的业务网络卡片会被写入到当前目录。

运行`composer card create`命令创建一个业务网络卡片，使用包含`Org1`和`Org2`的连接配置文件。你必须指定三个文件的路径，无论新建还是前面步骤定位的：
```bash
composer card create -p connection-org1.json -u PeerAdmin -c Admin@org1.example.com-cert.pem -k 78f2139bfcfc0edc7ada0801650ed785a11cfcdef3f9c36f3c8ca2ebfa00a59c_sk -r PeerAdmin -r ChannelAdmin
```
如果命令执行成功，一个叫`PeerAdmin@byfn-network-org1.card`的业务网络卡片会被写入到当前目录。

### ②步骤八：为Org2的Hyperledger Fabric管理员创建业务网络卡片

在这个步骤中你会为管理员创建业务网络卡片，用于将区块链业务网络部署到 Hyperledger Fabric网络。

运行`composer card create`命令创建一个业务网络卡片，使用包含`Org2`peer的连接配置文件。你必须指定三个文件的路径，无论新建还是前面步骤定位的(注意：*sk*文件不同)。
```bash
composer card create -p connection-org2-only.json -u PeerAdmin -c Admin@org2.example.com-cert.pem -k d4889cb2a32e167bf7aeced872a214673ee5976b63a94a6a4e61c135ca2f2dbb_sk -r PeerAdmin -r ChannelAdmin
```
如果命令执行成功，一个叫`PeerAdmin@byfn-network-org2-only.card`的业务网络卡片会被写入到当前目录。

运行`composer card create`命令创建一个业务网络卡片，使用包含`Org2`和`Org1`的连接配置文件。你必须指定三个文件的路径，无论新建还是前面步骤定位的：
```bash
composer card create -p connection-org2.json -u PeerAdmin -c Admin@org2.example.com-cert.pem -k d4889cb2a32e167bf7aeced872a214673ee5976b63a94a6a4e61c135ca2f2dbb_sk -r PeerAdmin -r ChannelAdmin
```
如果命令执行成功，一个叫`PeerAdmin@byfn-network-org2.card`的业务网络卡片会被写入到当前目录。

### ①步骤九：为Org1的Hyperledger Fabric管理员导入业务网络卡片

运行`composer card import`命令将包含`Org1`peer的业务网络卡片导入进钱包：
```bash
composer card import -f PeerAdmin@byfn-network-org1-only.card
```
如果这个命令执行成功，一个叫`PeerAdmin@byfn-network-org1-only`的业务网络卡片会被导入进钱包。

运行`composer card import`命令将包含`Org1`和`Org2`peer的业务网络卡片导入进钱包：
```bash
composer card import -f PeerAdmin@byfn-network-org1.card
```
如果命令执行成功，一个叫`PeerAdmin@byfn-network-org1`的业务网络卡片会被导入进钱包。

### ②步骤十：为Org2的Hyperledger Fabric管理员导入业务网络卡片

运行`composer card import`命令将包含`Org2`peer的业务网络卡片导入进钱包：
```bash
composer card import -f PeerAdmin@byfn-network-org2-only.card
```
如果这个命令执行成功，一个叫`PeerAdmin@byfn-network-org2-only`的业务网络卡片会被导入进钱包。

运行`composer card import`命令将包含`Org2`和`Org1`peer的业务网络卡片导入进钱包：
```bash
composer card import -f PeerAdmin@byfn-network-org2.card
```
如果命令执行成功，一个叫`PeerAdmin@byfn-network-org2`的业务网络卡片会被导入进钱包。

### ①步骤十一：为Org1的Hyperledger Fabric peer节点安装Hyperledger Composer运行时

使用你在步骤三中创建的连接profle，运行`composer runtime install`命令为所有的`Org1`peer节点安装Hyperledger Composer运行时：
```bash
composer runtime install -c PeerAdmin@byfn-network-org1-only -n tutorial-network
```

### ②步骤十二：为Org2的Hyperledger Fabric peer节点安装Hyperledger Composer运行时

使用你在步骤四中创建的连接profle，运行`composer runtime install`命令为所有的`Org2`peer节点安装Hyperledger Composer运行时：
```bash
composer runtime install -c PeerAdmin@byfn-network-org2-only -n tutorial-network
```

### ◎步骤十三：为业务网络定义背书策略

一个运行中的业务网络具有一个背书策略，它定义交易在提交到区块链之前那些组织必须对交易进行背书的规则。默认情况下，业务网络部署的背书策略是，一个交易被提交到区块链之前只有一个组织背书即可。

在现实世界的区块链业务网络中，多个组织会在交易提交到区块链之前对交易背书，所以默认背书策略不适合。相反，当启动一个业务网络时，你可以指定一个定制的背书策略。

你可以在Hyperledger Fabric文档，[背书策略](https://hyperledgercn.github.io/hyperledgerDocs/endorsement-policies_zh/)中，找到更多关于背书策略的信息。

*请注意，用于业务网络的背书策略必须是Hyperledger Fabric Node.js SDK使用的JSON格式。这是与Hyperledger Fabric CLI使用的简单背书策略不同的一种格式，你可以在Hyperledger Fabric文档中看到。*

创建一个叫`endorsement-policy.json`的背书策略，包含下列内容并保存到磁盘。你会在后续步骤中使用这个文件，所以记住存放位置！
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
你刚创建的背书策略指出，在业务网络中，交易在提交到区块链之前必须同时得到`Org1`和`Org2`的背书。如果`Org1`或`Org2`没有对交易背书，或不同意交易结果，那么交易会被业务网络拒绝。

### ◎步骤十四：理解和选择业务网络管理员

当一个业务网络启动时，业务网络必须配置一组初始参与者。这些参与者负责业务网络的引导和加载其他参与者进入业务网络。在Hyperledger Composer中，我们叫这些初始参与者为业务网络管理员。

在我们的业务网络中，组织`Org1`和`Org2`具有相同权限。每个组织都为业务网络提供了一个业务网络管理员，这些业务网络管理员会加载他们组织中的其他参与者进入业务网络。`Org1`的业务网络管理员是Alice，而`Org2`的业务网络管理员是Bob。

当业务网络启动时，所有业务网络管理员的证书(身份的公开部分)都必须传送到组织，来执行启动业务网络的命令。当业务网络启动后，所有的业务网络管理员可以使用他们的身份与业务网络进行交互。

你可以在[部署业务网络](business-network_bnd-deploy.md)中找到更多关于业务网络管理员的信息。

### ①步骤十五：为Org1获取业务网络管理员证书

运行`composer identity request`命令获取Alice的证书，以便`Org1`作为业务网络管理员使用：
```bash
composer identity request -c PeerAdmin@byfn-network-org1-only -u admin -s adminpw -d alice
```
这个命令的`-u admin`和`-s adminpw`选项需要与Hyperledger Fabric CA (Certificate Authority)注册的默认用户一致。

这个证书会被保存到当前目录的`alice`子目录中。创建了三个证书文件，但只有两个重要。它们是`admin-pub.pem`，证书(包括公钥)，和`admin-priv.pem`，私钥。只有文件`admin-pub.pem`适合与其他组织共享。文件`admin-priv.pem`必须秘密保存，它可以代表组织签署交易。

### ②步骤十六：为Org2获取业务网络管理员证书

运行`composer identity request`命令获取Bob的证书，以便`Org2`作为业务网络管理员使用：
```bash
composer identity request -c PeerAdmin@byfn-network-org2-only -u admin -s adminpw -d bob
```
这个命令的`-u admin`和`-s adminpw`选项需要与Hyperledger Fabric CA (Certificate Authority)注册的默认用户一致。

这个证书会被保存到当前目录的`bob`子目录中。创建了三个证书文件，但只有两个重要。它们是`admin-pub.pem`，证书(包括公钥)，和`admin-priv.pem`，私钥。只有文件`admin-pub.pem`适合与其他组织共享。文件`admin-priv.pem`必须秘密保存，它可以代表组织签署交易。

### ①步骤十七：启动业务网络

运行`composer network start`命令启动业务网络。只有`Org`需要执行这个操作。这个命令使用步骤十三创建的`endorsement-policy.json`文件，以及步骤十五(Alice)和步骤十六(Bob)创建的`admin-pub.pem`文件，你必须确保这个命令能访问这些文件：
```bash
composer network start -c PeerAdmin@byfn-network-org1 -a tutorial-network@0.0.1.bna -o endorsementPolicyFile=endorsement-policy.json -A alice -C alice/admin-pub.pem -A bob -C bob/admin-pub.pem
```
当这个命令一结束，业务网络就启动了。Alice和Bob就可以访问业务网络，开始建立业务网络，并从各自的组织中加载其他参与者。然而，无论Alice还是Bob都必须先使用前面步骤创建的证书来创建业务网络卡片，之后才能访问业务网络。

### ①步骤十八：为了Org1访问业务网络而创建业务网络卡片

为了`Org1`的业务网络管理员Alice访问业务网络，运行`composer card create`命令创建一个业务网络卡片：
```bash
composer card create -p connection-org1.json -u alice -n tutorial-network -c alice/admin-pub.pem -k alice/admin-priv.pem
```
运行`composer card import`命令导入你刚创建的业务网络卡片：
```bash
composer card import -f alice@tutorial-network.card
```
运行`composer network ping`命令测试到区块链业务网络的连接：
```bash
composer network ping -c alice@tutorial-network
```
如果名称执行成功，你可以在命令输出中看到完全限定参与者身份`org.hyperledger.composer.system.NetworkAdmin#alice`。你现在可以使用这个业务网络卡片去与区块链业务网络交互，并加载你组织中的其他参与者。

### ②步骤十九：为了Org1访问业务网络而创建业务网络卡片

为了`Org2`的业务网络管理员Bob访问业务网络，运行`composer card create`命令创建一个业务网络卡片：
```bash
composer card create -p connection-org2.json -u bob -n tutorial-network -c bob/admin-pub.pem -k bob/admin-priv.pem
```
运行`composer card import`命令导入你刚创建的业务网络卡片：
```bash
composer card import -f bob@tutorial-network.card
```
运行`composer network ping`命令测试到区块链业务网络的连接：
```bash
composer network ping -c bob@tutorial-network
```
如果名称执行成功，你可以在命令输出中看到完全限定参与者身份`org.hyperledger.composer.system.NetworkAdmin#bob`。你现在可以使用这个业务网络卡片去与区块链业务网络交互，并加载你组织中的其他参与者。

### ◎结论
在这个教程中你已经看到了怎样使用所有必要信息配置Hyperledger Composer去连接到跨组织的Hyperledger Fabric网络，以及在一个跨组织的Hyperledger Fabric网络中怎样部署一个区块链业务网络。
