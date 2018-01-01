# Hyperledger Composer网络更新

`composer network update`实用程序用于将已部署业务网络档案从本地磁盘更新到Hyperledger Fabric运行时。
```bash
composer network update -a <business-network-archive> -c <card-name>
```

业务网络定义必须先被部署到Fabric。业务网络定义在相同的链码容器内被替换。

### 选项
```bash
Options:
  --help              Show help  [boolean]
  -v, --version       Show version number  [boolean]
  --card, -c          The card to use to update the network  [string] [required]
  --archiveFile , -a  The business network archive file [string] [required]
```

## 输出示例
```bash
composer network update -a digitalPropertyNetwork.bna -c admin@digitalPropertyNetwork
Deploying business network from archive: digitalPropertyNetwork.bna
Business network definition:
    Identifier: digitalproperty-network@0.0.1
    Description: Digital Property Network
Updating business network definition. This may take a few seconds...

Command succeeded
```
