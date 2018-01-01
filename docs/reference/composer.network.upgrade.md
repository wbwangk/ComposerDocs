# Hyperledger Composer网络升级

`composer network upgrade`实用程序用于将已部署业务网络档案从本地磁盘更新到Hyperledger Fabric运行时。
```bash
composer network upgrade -n <business-network-archive> -p <connection-profile-Name> -i <upgrade-Id> -s <upgrade-Secret>
```

`composer network upgrade`升级指定业务网络的Hyperledger Composer运行时，以使用新的微型版本。在运行`composer network upgrade`命令之前，必须先使用`composer runtime install`命令将一个Hyperledger Composer运行的新版本部署到区块链节点。

*请注意*：`composer network upgrade`仅适用于Hyperledger Composer运行时的微版本之间的升级。微版本被定义为发布版本的第三个十进制数，例如，发布版本0.9.2，主版本是0，次版本是9，微版本是2。

### 选项
```bash
composer network upgrade [options]

Options:
  --help                       Show help  [boolean]
  -v, --version                Show version number  [boolean]
  -c, --card                   The business network card defining the network to upgrade. [string]
```
