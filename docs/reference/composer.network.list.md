# Hyperledger Composer网络列表

`composer network list`实用程序用于连接到业务网络并获取元数据和资产信息。
```bash
composer network list -c admin@tutorial-network
```

### 选项
```
Options:
  --help          Show help  [boolean]
  -v, --version   Show version number  [boolean]
  --registry, -r  List specific registry  [string]
  --asset, -a     List specific asset  [string]
  --card, -c      The card name used to list the network  [string]
```

## 输出示例
```bash
composer network list -c admin@tutorial-network

✔ List business network digitalproperty-network
name:       digitalproperty-network
models:
  - org.hyperledger.composer.system
  - net.biz.digitalPropertyNetwork
scripts:
  - lib/DigitalLandTitle.js
registries:
  net.biz.digitalPropertyNetwork.LandTitle:
    id:           net.biz.digitalPropertyNetwork.LandTitle
    name:         Asset registry for net.biz.digitalPropertyNetwork.LandTitle
    registryType: Asset
    assets:
      LID:1148:
        $class:      net.biz.digitalPropertyNetwork.LandTitle
        titleId:     LID:1148
        owner:       resource:net.biz.digitalPropertyNetwork.Person#PID:1234567890
        information: A nice house in the country
        forSale:     true
      LID:6789:
        $class:      net.biz.digitalPropertyNetwork.LandTitle
        titleId:     LID:6789
        owner:       resource:net.biz.digitalPropertyNetwork.Person#PID:1234567890
        information: A small flat in the city
  net.biz.digitalPropertyNetwork.SalesAgreement:
    id:           net.biz.digitalPropertyNetwork.SalesAgreement
    name:         Asset registry for net.biz.digitalPropertyNetwork.SalesAgreement
    registryType: Asset
  net.biz.digitalPropertyNetwork.Person:
    id:           net.biz.digitalPropertyNetwork.Person
    name:         Participant registry for net.biz.digitalPropertyNetwork.Person
    registryType: Participant
    assets:
      PID:1234567890:
        $class:    net.biz.digitalPropertyNetwork.Person
        personId:  PID:1234567890
        firstName: Fred
        lastName:  Bloggs

Command succeeded
```
