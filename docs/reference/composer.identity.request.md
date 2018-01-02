# Composer身份请求

`composer identity request`用于请求一个身份的证书和私钥。

## 句法
```bash
composer identity request

Options:
  --help              Show help  [boolean]
  -v, --version       Show version number  [boolean]
  --card, -c          Name of the network card to use  [string] [required]
  --user, -u          The name of the identity for the card (if different from the card)  [string]
  --enrollSecret, -s  The enrollment secret of the user (if different from the card)  [string]
  --path, -d          path where to store the certificates and key  [string]
```
注意：当加上`-u`参数后，会覆盖卡片中的身份定义，这意味着仅适用了卡片中的连接配置文件。

## 选项

`--card, -c`

指定连接到Comopser使用的业务网络卡片。

`--user, -u`
业务网络卡片中含有身份和连接配置文件，如果想获取其他身份的证书和私钥（仅使用连接配置文件），就是使用这个参数。

`--enrollSecret, -s`
与`-u`参数配合使用，表示身份的登记密码。

`--path, -d`
是一个目录。获取的证书和私钥会保存到这个目录。

## 例子
这个例子来自[为多个组织部署Hyperledger Fabric](../orials_deploy-to-fabric-multi-org/#1org1_1)

运行`composer identity request`命令获取Alice的证书，以便组织`Org1`作为业务网络管理员使用：
```bash
composer identity request -c PeerAdmin@byfn-network-org1-only -u admin -s adminpw -d alice
```
这个命令的`-u admin`和`-s adminpw`选项需要与Hyperledger Fabric CA (Certificate Authority)注册的默认用户一致。

这个证书会被保存到当前目录的`alice`子目录中。创建了三个证书文件，但只有两个重要。它们是`admin-pub.pem`，证书(包括公钥)，和`admin-priv.pem`，私钥。只有文件`admin-pub.pem`适合与其他组织共享。文件`admin-priv.pem`必须秘密保存，它可以代表组织签署交易。
