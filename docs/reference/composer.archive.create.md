# Hyperledger Composer存档创建

`composer archive create`实用程序用于从目录的内容创建业务网络档案。

要从当前“工作”目录中的源文件（即业务网络定义项目文件）创建档案：
```bash
composer archive create -a <business-network-archive>
```

或者

指定路径（到源业务网络定义和目标档案文件（.bna文件）的目录）：
```bash
composer archive create --sourceType dir --sourceName <dirpath> -a digitalproperty-network.bna
```

### 选项
```bash
composer archive create --archiveFile digitialPropertyNetwork.zip --sourceType module --sourceName digitalproperty-network

Options:
  --help             Show help  [boolean]
  -v, --version      Show version number  [boolean]
  --archiveFile, -a  Business network archive file name. Default is based on the Identifier of the BusinessNetwork  [string]
  --sourceType, -t   The type of the input containg the files used to create the archive [ module | dir ]  [required]
  --sourceName, -n   The Location to create the archive from e.g. NPM module directory or Name of the npm module to use  [required]
Only one of either inputDir or moduleName must be specified.
```

## 命令和输出的示例

```bash
$ pwd
/Users/dselman/dev/temp

composer archive create --sourceType dir --sourceName . -a dist/digitalproperty-network.bna

Creating Business Network Archive
Looking for package.json of Business Network Definition in /Users/dselman/dev/temp/dist

Description:Digital Property Network
Name:digitalproperty-network
Identifier:digitalproperty-network@0.0.1

Written Business Network Definition Archive file to digitalproperty-network@0.0.1.bna
Command completed successfully.
```
