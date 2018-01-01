# Hyperledger Composer网络部署

`composer network deploy`实用程序用于将业务网络档案从本地磁盘部署到Hyperledger Fabric v1.0网络。在使用此命令之前，请阅读[部署和更新业务网络](business-network_bnd-deploy.html)文档。
```bash
composer network deploy -a digitalPropertyNetwork.bna -A admin -S -c PeerAdmin@hlfv1 -f admincard
```

### 选项
```bash
composer network deploy [options]

Options:
  --help                             Show help  [boolean]
  -v, --version                      Show version number  [boolean]
  --archiveFile, -a                  The business network archive file name  [string] [required]
  --loglevel, -l                     The initial loglevel to set  [choices: "INFO", "WARNING", "ERROR", "DEBUG"]
  --option, -o                       Options that are specific specific to connection. Multiple options are specified by repeating this option  [string]
  --optionsFile, -O                  A file containing options that are specific to connection  [string]
  --networkAdmin, -A                 The identity name of the business network administrator  [string]
  --networkAdminCertificateFile, -C  The certificate of the business network administrator  [string]
  --networkAdminEnrollSecret, -S     Use enrollment secret for the business network administrator  [boolean]
  --card, -c                         The cardname to use to deploy the network  [string]
  --file, -f                         File name of the card to be created  [string]
```

## 输出示例
```bash
composer network deploy -a digitalPropertyNetwork.bna -A admin -S -c PeerAdmin@hlfv1 -f admincard

Deploying business network from archive digitalPropertyNetwork.bna
Business network definition:
    Identifier: digitalproperty-network@0.0.1
    Description: Digital Property Network
Deploying business network definition. This may take a minute...
Command completed successfully.
```

## Hyperledger Fabric背书策略

`--option, -o`选项和`--optionsFile, -O`选项允许发送特定于连接的信息。Hyperledger Fabric v1.0背书策略可以通过几种方式使用`-o`和`-O`选项发送。

- 使用该`-o`选项，背书策略可以作为单行JSON字符串发送，如下所示：
```bash
composer network deploy -o endorsementPolicy='{"identities": [.... }'
```

- 使用`-o`选项，背书策略可以作为文件路径发送，如下所示：
```bash
composer network deploy -o endorsementPolicyFile=/path/to/file/endorsementPolicy.json
```

在这种情况下，背书策略文件应该遵循以下格式：
```json
{"identities":[...],
    "policy": {...}}
```

- 使用该`-O`选项，背书策略可以作为文件路径发送，如下所示：
```bash
composer network deploy -O /path/to/file/options.json
```

  在这种情况下，选项文件应该遵循以下格式：
```json
        {"endorsementPolicy": {"Identities": [...].
            "policy: {...}"
          },
          "someOtherOption": "A Value"
        }
```

有关编写Hyperledger Fabric背书策略的更多信息，请参阅[Hyperledger结构Node.js SDK文档](https://fabric-sdk-node.github.io/global.html#Policy)。
