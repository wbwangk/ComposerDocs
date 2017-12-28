# Hyperledger Composer REST服务器

Hyperledger Composer REST服务器，即`composer-rest-server`，可用于从已部署的区块链业务网络生成REST API，该网络可由HTTP或REST客户端轻松使用。

## 使用环境变量配置REST服务器

可以使用环境变量来配置REST服务器，而不是通过命令行提供配置选项。REST服务器支持以下环境变量：

1. `COMPOSER_CARD`

你可以使用`COMPOSER_CARD`环境变量来指定发现业务网络卡片的名称，REST服务器用它来连接到业务网络的。

例如：
```
COMPOSER_CARD=admin@my-network
```

2. `COMPOSER_NAMESPACES`

你可以使用`COMPOSER_NAMESPACES`环境变量来指定REST服务器是否应使用命名空间生成REST API。有效值是`always`，`required`和`never`。

例如：
```
COMPOSER_NAMESPACES=never
```

3. `COMPOSER_AUTHENTICATION`

你可以使用`COMPOSER_AUTHENTICATION`环境变量来指定REST服务器是否应启用REST API身份验证。有效值是`true`和`false`。

例如：
```
COMPOSER_AUTHENTICATION=true
```

有关更多信息，请参阅[启用REST服务器的身份验证](integrating_enabling-rest-authentication.md)。

4. `COMPOSER_MULTIUSER`

你可以使用`COMPOSER_MULTIUSER`环境变量来指定REST服务器是否应启用多用户模式。有效值是`true`和`false`。

例如：
```
COMPOSER_MULTIUSER=true
``

有关更多信息，请参阅[为REST服务器启用多用户模式](integrating_enabling-multiuser.md)。

5. `COMPOSER_PROVIDERS`

你可以使用`COMPOSER_PROVIDERS`环境变量来指定REST服务器应用来验证REST API的客户端的Passport策略。

例如：
```
   COMPOSER_PROVIDERS='{
     "github": {
       "provider": "github",
       "module": "passport-github",
       "clientID": "REPLACE_WITH_CLIENT_ID",
       "clientSecret": "REPLACE_WITH_CLIENT_SECRET",
       "authPath": "/auth/github",
       "callbackURL": "/auth/github/callback",
       "successRedirect": "/",
       "failureRedirect": "/"
     }
   }'
```

6. `COMPOSER_DATASOURCES`

你可以使用`COMPOSER_DATASOURCES`环境变量来指定所选LoopBack连接器所需的LoopBack数据源和连接信息。

例如：
```
   COMPOSER_DATASOURCES='{
     "db": {
       "name": "db",
       "connector": "mongodb",
       "host": "mongo"
     }
   }'
```

7. `COMPOSER_TLS`

你可以使用`COMPOSER_TLS`环境变量来指定REST服务器是否应启用HTTPS和TLS。有效值是`true`和`false`。

例如：
```
   COMPOSER_TLS=true
```

有关更多信息，请参阅[使用HTTPS和TLS保护REST服务器](integrating_securing-the-rest-server.md)。

8. `COMPOSER_TLS_CERTIFICATE`

你可以使用`COMPOSER_TLS_CERTIFICATE`环境变量来指定启用HTTPS和TLS时REST服务器应使用的证书文件。

例如：
```
   COMPOSER_TLS_CERTIFICATE=/tmp/cert.pem
```

9. `COMPOSER_TLS_KEY`

你可以使用`COMPOSER_TLS_KEY`环境变量来指定在启用HTTPS和TLS时REST服务器应使用的私钥文件。

例如：
```
   COMPOSER_TLS_KEY=/tmp/key.pem
```
