# 使用HTTPS和TLS来保护REST服务器

在生产环境中部署Hyperledger Composer REST服务器时，应将REST服务器配置为使用HTTPS和TLS（传输层安全性）进行保护。一旦REST服务器配置了HTTPS和TLS，在REST服务器和所有REST客户端之间传输的所有数据都将被加密。

您必须提供证书和私钥对才能配置REST服务器。REST服务器包含可用于轻松起步的示例证书和私钥对，但是此配置仅建议在初始开发期间使用。不要在生产环境中使用示例证书和私钥对。

## 使用示例证书和私钥对启用HTTPS和TLS

您可以在命令行中用`-t`参数使用示例证书和私钥对来启用HTTPS和TLS ：
```bash
composer-rest-server -c alice1@my-network -t
```

或者，可以用环境变量`COMPOSER_TLS`来使用示例证书和私钥对启用HTTPS和TLS ：
```bash
export COMPOSER_TLS=true
composer-rest-server -c alice1@my-network
```

成功启用HTTPS和TLS后，您将看到REST服务器的输出指定了一个`https://`URL而不是`http://`URL：
```
Web server listening at: https://localhost:3000
Browse your REST API at https://localhost:3000/explorer
```

只有在初始开发过程中才推荐使用此配置。对于测试，QA或生产部署，您应提供自己的证书和私钥来启用HTTPS和TLS。

## 通过提供证书和私钥对来启用HTTPS和TLS

您可以通过提供自己的证书和私钥对来启用HTTPS和TLS。证书和私钥对必须作为两个单独的文件以PEM格式提供。这些文件必须在运行REST服务器的系统的文件系统上可用，并且REST服务器必须具有对这些文件的读取权限。

您可以通过在命令行中使用“-e”（证书文件）和“-k”（私钥文件）参数来配置REST服务器以使用你的证书和私钥对文件：
```bash
composer-rest-server -c alice1@my-network -t -e /tmp/cert.pem -k /tmp/key.pem
```

或者，可以使用`COMPOSER_TLS_CERTIFICATE`和`COMPOSER_TLS_KEY`环境变量将REST服务器配置为使用你的证书和私钥对文件：
```bash
export COMPOSER_TLS=true
export COMPOSER_TLS_CERTIFICATE=/tmp/cert.pem
export COMPOSER_TLS_KEY=/tmp/key.pem
composer-rest-server -c alice1@my-network
```
