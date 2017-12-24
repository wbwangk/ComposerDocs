# 将现有身份绑定到参与者

现有的身份可以通过API或命令行发放给参与者。一旦现有的身份被绑定，参与者就可以使用该身份在该参与者的上下文中与业务网络进行交互。

使用{[site.data.conrefs.hlf_full}}时，可以绑定使用{[site.data.conrefs.hlf_full}}证书颁发机构（CA）创建的现有证书，或使用其他工具如`cryptogen`。现有证书必须有效用于在{[site.data.conrefs.hlf_full}}网络上提交交易。

## 在你开始之前

在执行这些步骤之前，您必须已将参与者添加到参与者库。您必须拥有PEM格式的现有证书才能绑定到参与者。现有身份（无论是使用命令行还是使用下面的JavaScript API）的**绑定者**必须具有ACL，允许他们绑定Hyperledger Composer中的身份（与参与者关联）。

下面的过程显示了一个使用以下数字财产范例业务网络定义的参与者模型的示例：[digitalproperty-network](https://www.npmjs.com/package/digitalproperty-network)

```
namespace net.biz.digitalPropertyNetwork

participant Person identified by personId {
  o String personId
  o String firstName
  o String lastName
}
```

该示例假定该`net.biz.digitalPropertyNetwork#mae@biznet.org`参与者的实例已经被创建并被放入参与者库中。

## 过程

1. 连接到业务网络并将现有身份绑定到参与者
   - JavaScript API
```javascript
  const BusinessNetworkConnection = require('composer-client').BusinessNetworkConnection;
  let businessNetworkConnection = new BusinessNetworkConnection();
  let certificate = `-----BEGIN CERTIFICATE-----
    MIIB8DCCAZegAwIBAgIURanHh55fqrUecvHNHtcMKiHJRkwwCgYIKoZIzj0EAwIw
    czELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNh
    biBGcmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHDAaBgNVBAMT
    E2NhLm9yZzEuZXhhbXBsZS5jb20wHhcNMTcwNzI3MTc0MzAwWhcNMTgwNzI3MTc0
    MzAwWjAQMQ4wDAYDVQQDEwVhZG1pbjBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IA
    BAANIGFIrXXr5+h0NfUNJhx5YFQ4w6r182eZYRhc9KvYQhYo5D0ZbecfR9sGX2b6
    0aW+C7bUaXc6DU3pJSD4fNijbDBqMA4GA1UdDwEB/wQEAwIHgDAMBgNVHRMBAf8E
    AjAAMB0GA1UdDgQWBBRwuAyWrGlzVQFqRf0OqoTNuoq7QDArBgNVHSMEJDAigCAZ
    q2WruwSAfa0S5MCpqqZknnCGjjq9AhejItieR+GmrjAKBggqhkjOPQQDAgNHADBE
    AiBcj/JvxmKHel4zQ3EmjITEFhdYku5ijIZEDuR5v9HK3gIgTUbVEfq3MuasVZKx
    rkM5DH3e5ECM7T+T1Ovr+1AK6bs=
    -----END CERTIFICATE-----`
  return businessNetworkConnection.connect('admin@digitalPropertyNetwork')
      .then(() => {
          return businessNetworkConnection.bindIdentity('net.biz.digitalPropertyNetwork.Person#mae@biznet.org', certificate)
      })
      .then(() => {
          return businessNetworkConnection.disconnect();
      })
      .catch((error) => {
          console.error(error);
          process.exit(1);
      });
```

- 命令行
  ```bash
  composer identity bind -c admin@digitalPropertyNetwork -a "resource:net.biz.digitalPropertyNetwork.Person#mae@biznet.org"
  ```

1. 作为参与者，测试与业务网络的连接
   - JavaScript API
```javascript
  const BusinessNetworkConnection = require('composer-client').BusinessNetworkConnection;
  let businessNetworkConnection = new BusinessNetworkConnection();
  return businessNetworkConnection.connect('admin@digitalPropertyNetwork')
      .then(() => {
          return businessNetworkConnection.ping();
      })
      .then((result) => {
          console.log(`participant = ${result.participant ? result.participant : '<no participant found>'}`);
          return businessNetworkConnection.disconnect();
      })
      .catch((error) => {
          console.error(error);
          process.exit(1);
      });
```

- 命令行
  ```bash
  composer network ping -c admin@digitalPropertyNetwork
  ```

参与者ID将被打印到控制台，并且应该与`composer identity bind`命令中指定的参与者ID匹配。
