# Hyperledger Composer运行时安装

`composer runtime install`命令用于在你要连接的区块链网络的Hyperledger Fabric peer端上安装Hyperledger Composer运行时。该命令必须在`composer network start`命令之前运行。

*请注意*：该`--businessNetworkName, -n`选项**必须**包含与打算在Hyperledger Fabric peer上运行的业务网络名称相同的名称。只有名称与`--businessNetworkName, -n`命令中给出的选项匹配的业务网络才能成功运行。
```bash
composer runtime install -n <businessNetworkName> -c <peer-admin-card>
```

### 选项
```bash
composer runtime install [options]

Options:
  --help                     Show help  [boolean]
  -v, --version              Show version number  [boolean]
  --option, -o               Options that are specific specific to connection. Multiple options are specified by repeating this option  [string]
  --optionsFile, -O          A file containing options that are specific to connection  [string]
  --card, -c                 The cardname to use to install the network  [string] [required]
  --businessNetworkName, -n  The business network name  [string] [required]
```
