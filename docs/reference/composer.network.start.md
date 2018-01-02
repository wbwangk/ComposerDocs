# Hyperledger Composer网络启动

`composer network start`实用程序用于将业务网络档案从本地磁盘部署到Hyperledger Fabric v1.0网络。在使用此命令之前，请阅读[部署和更新业务网络](../business-network_bnd-deploy.md)主题。

*请注意*：你**必须**先使用`composer runtime install`命令将Hyperledger Composer运行时安装到Hyperledger Fabric peer节点。`composer runtime install`命令中指定的业务网络名称必须与`composer network start`命令中指定的业务网络名称相同。

```bash
composer network start -a <business-network-archive> -A <admin-name> -S adminpw -c <business-network-card> -f <name-of-admin-card>
```

### 选项
```bash
composer network start [options]

Options:
  --help                             Show help  [boolean]
  -v, --version                      Show version number  [boolean]
  --archiveFile, -a                  The business network archive file name  [string] [required]
  --loglevel, -l                     The initial loglevel to set  [choices: "INFO", "WARNING", "ERROR", "DEBUG"]
  --option, -o                       Options that are specific specific to connection. Multiple options are specified by repeating this option  [string]
  --optionsFile, -O                  A file containing options that are specific to connection  [string]
  --networkAdmin, -A                 The identity name of the business network administrator  [string]
  --networkAdminCertificateFile, -C  The certificate of the business network administrator  [string]
  --networkAdminEnrollSecret, -S     Use enrollment secret for the business network administrator  [string]
  --card, -c                         The cardname to use to start the network  [string]
  --file, -f                         File name of the card to be created  [string]

```

## Hyperledger Fabric背书策略

`--option, -o`选项和`--optionsFile, -O`选项允许发送特定于连接的信息。Hyperledger Fabric背书策略可以通过几种方式使用`-o`和`-O`选项发送。

- 使用`-o`选项，背书策略可以作为单行JSON字符串发送，如下所示：
```bash
composer network start -o endorsementPolicy='{"identities": [.... }'
```

- 使用`-o`选项，背书策略可以作为文件路径发送，如下所示：
```bash
composer network start -o endorsementPolicyFile=/path/to/file/endorsementPolicy.json
```

  在这种情况下，背书策略文件应该遵循以下格式：
```json
  {"identities":[...],
      "policy": {...}}
```

- 使用`-O`选项，背书策略可以作为文件路径发送，如下所示：
```bash
composer network start -O /path/to/file/options.json
```

  在这种情况下，选项文件应该遵循以下格式：
```json
        {"endorsementPolicy": {"Identities": [...].
            "policy: {...}"
          },
          "someOtherOption": "A Value"
        }
```

有关编写Hyperledger Fabric认可策略的更多信息，请参阅[Hyperledger结构Node.js SDK文档](https://fabric-sdk-node.github.io/global.html#Policy)。
