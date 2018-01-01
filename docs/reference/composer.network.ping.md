# Hyperledger Composer网络Ping

`composer network ping`实用程序用于验证与部署到Hyperledger Fabric业务网络的连接。请注意，如果已为参与者颁发了身份，则ping还会返回用于连接到网络的身份的参与者信息。
```bash
composer network ping --card admin@tutorial-network
```

### 选项
```bash
Options:
  --help         Show help  [boolean]
  -v, --version  Show version number  [boolean]
  --card, -c     The cardname to use to ping the network  [string]
```

## 输出示例
```bash
composer network ping --card admin@tutorial-network
The connection to the network was successfully tested: tutorial-network
    version: 0.15.0-20171108090428
    participant: org.hyperledger.composer.system.NetworkAdmin#admin

Command succeeded
```
