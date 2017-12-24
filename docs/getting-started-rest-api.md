# 生成一个REST API

## 安装REST服务器

Hyperledger Composer REST服务器可以使用npm或Docker进行安装。

要使用npm进行安装，请运行以下命令：
```bash
npm install -g composer-rest-server
```

要使用Docker安装REST服务器，请参阅[部署REST服务器](https://hyperledger.github.io/composer/stable/integrating/deploying-the-rest-server.html)。

## 运行REST服务器

Hyperledger Composer包含独立的Node.js进程，将业务网络公开为REST API。LoopBack框架用于生成由Swagger文档描述的Open API。

要启动REST服务器，只需输入：
```bash
composer-rest-server
```

然后您将被要求输入关于你的业务网络的一些简单的细节。下面显示了使用已部署的业务网络的一个示例。
```
? Enter the name of the business network card to use: admin@basic-sample-network
? Specify if you want namespaces in the generated REST API: always use namespaces
? Specify if you want to enable authentication for the REST API using Passport: No
? Specify if you want to enable event publication over WebSockets: Yes
? Specify if you want to enable TLS security for the REST API: No

To restart the REST server using the same options, issue the following command:
   composer-rest-server -c admin@basic-sample-network -n always -w true

Discovering types from business network definition ...
Discovered types from business network definition
Generating schemas for all types in business network definition ...
Generated schemas for all types in business network definition
Adding schemas for all types to Loopback ...
Added schemas for all types to Loopback
Web server listening at: http://localhost:3000
Browse your REST API at http://localhost:3000/explorer
```

#### `composer-rest-server`命令

该`composer-rest-server`命令有多个选项用于定义安全性和身份认证：
```bash
Options:
  -c, --card            The name of the business network card to use  [string]
  -n, --namespaces      Use namespaces if conflicting types exist  [string] [choices: "always", "required", "never"] [default: "always"]
  -p, --port            The port to serve the REST API on  [number]
  -a, --authentication  Enable authentication for the REST API using Passport  [boolean] [default: false]
  -m, --multiuser       Enable multiple user and identity management using wallets (implies -a)  [boolean] [default: false]
  -w, --websockets      Enable event publication over WebSockets  [boolean] [default: true]
  -t, --tls             Enable TLS security for the REST API  [boolean] [default: false]
  -e, --tlscert         File containing the TLS certificate  [string] [default: "/usr/local/lib/node_modules/composer-rest-server/cert.pem"]
  -k, --tlskey          File containing the TLS private key  [string] [default: "/usr/local/lib/node_modules/composer-rest-server/key.pem"]
  -h, --help            Show help  [boolean]
  -v, --version         Show version number  [boolean]
```

#### 看看生成的API

启动浏览器并转到给定的URL（[http：//0.0.0.0:3000/explorer](http://0.0.0.0:3000/explorer)）。你会看到类似这样的屏幕。

![回送1](https://hyperledger.github.io/composer/stable/assets/img/tutorials/developer/lb_explorer.png)

## 更新REST服务器

在更新业务网络定义之后，可以更新REST服务器以生成反映业务网络定义更新的新API。

要更新REST服务器，首先必须使用`ctrl-C`停止REST服务器。然后可以使用`composer-rest-server`重新启动REST服务器。

# 总结

在Hyperledger Composer运行时之上使用Loopback框架使我们能够基于已部署的业务网络模型生成特定于业务领域的REST API！
