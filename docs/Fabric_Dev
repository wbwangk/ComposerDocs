# Fabric CA User’s Guide
[官方原文](http://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#table-of-contents)  
它提供的功能：
- 注册身份(identity)，或在用户注册时连接到LDAP  
- 发放注册证书(ECert)  
- 证书更新或撤销  

Hyperledger Fabric CA由一个服务器和一个客户端组件构成。  
下图展示了CA与Hyperledger Fabric整体架构的关系。  
![](http://hyperledger-fabric-ca.readthedocs.io/en/latest/_images/fabric-ca.png)  

有两种方式与Hyperledger Fabric CA交互：通过Hyperledger Fabric CA客户端或通过Fabric SDK。所有与Hyperledger Fabric CA服务器的通信是通过REST API。`fabric-ca/swagger/swagger-fabric-ca.json`是swagger格式的REST API文档，可以通过在线工具[http://editor2.swagger.io](http://editor2.swagger.io/)查看。  
可以搭建Hyperledger Fabric CA集群，在集群下，所有服务器共享同一个数据库，数据库中存有身份和证书。如果配置了LDAP，身份(identity)信息就保存在LDAP中，而不是数据库。  
一个服务器可以包含多个CA。每个CA要么是根CA，要么是中间CA。每个中间CA有一个父CA，父CA要么是根CA要么是另一个中间CA。  

[编写首个应用](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E7%BC%96%E5%86%99%E9%A6%96%E4%B8%AA%E5%BA%94%E7%94%A8)中通过命令启动了几个容器：
```
$ cd /opt/fabric-samples/fabcar
$ ./startFabric.sh
```
其中的`ca.example.com`就是一个CA容器，可以进入容器去看看($$表示在容器内的命令)：
```
$ docker exec -it ca.example.com bash
$$ whereis fabric-ca-server
fabric-ca-server: /usr/local/bin/fabric-ca-server
$$ ps -ef | grep fabric-ca-server
root         7     1  0 00:42 ?        00:00:00 fabric-ca-server start -b admin:adminpw -d
```
用`ps`命令查看CA的守护进程，可以看到它的启动命令是：
```
fabric-ca-server start -b admin:adminpw -d
```
`-b`参数指出了注册(enrollement)ID和密码。  
`fabric-ca-server-config.yaml`是Fabric CA的默认配置文件，可以通过find命令查找它的位置：  
```
$$ find / -name fabric-ca-server-config.yaml
/etc/hyperledger/fabric-ca-server/fabric-ca-server-config.yaml
```
#### CA相关环境变量
```
$$ env
FABRIC_CA_SERVER_CA_NAME=ca.example.com
FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/4239aa0dcd76daeeb8ba0cda701851d14504d31aad1b2ddddbac6a57365e497c_sk
FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem
FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
```
## Fabric CA CLI
命令帮助信息：[Server Command Line](http://hyperledger-fabric-ca.readthedocs.io/en/latest/servercli.html)、[Client Command Line](http://hyperledger-fabric-ca.readthedocs.io/en/latest/clientcli.html)  

### Fabric CA客户端
