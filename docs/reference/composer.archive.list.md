# Hyperledger Composer档案列表

`composer archive list`实用程序用于验证磁盘上业务网络档案的结构并打印元数据。

```bash
composer archive list -a <business-network-archive file>
```

### 选项
```bash
--help             Show help  [boolean]
  -v, --version      Show version number  [boolean]
  --archiveFile, -a  Business network archive file name.  [string]
```
## 输出示例
```bash
composer archive list -a digitalPropertyNetwork.bna
Listing Business Network Archive from digitalPropertyNetwork.bna
Identifier:digitalproperty-network@0.0.1
Name:digitalproperty-network
Version:0.0.1

Command succeeded
```
