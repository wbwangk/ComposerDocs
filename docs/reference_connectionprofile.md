# 连接配置文件

Hyperledger Composer使用连接配置文件连接到运行时。

## 创建连接配置文件

1.创建一个名为`connection.json`的新文件，其中包含Hyperledger Fabric v1.0的以下信息。}

为Hyperledger Fabric v1.0创建连接配置文件，使用以下格式：
```json
    {
        "type": "hlfv1",
        "orderers": [
           { "url" : "grpc://localhost:7050" }
        ],
        "ca": { "url": "http://localhost:7054",
                "name": "ca.org1.example.com"
        },
        "peers": [
            {
                "requestURL": "grpc://localhost:7051",
                "eventURL": "grpc://localhost:7053"
            }
        ],
        "keyValStore": "${HOME}/.composer-credentials",
        "channel": "composerchannel",
        "mspID": "Org1MSP",
        "timeout": "300"
    }

If you are connecting to Hyperledger Fabric v1.0 and are not using TLS or if you don't need the trustedRoots and verify options of the Certificate Authority definition you can use the following simplified connection profile:

_Please note: The simplified version of the connection profile will only work if the relevant certificate authority has no name defined. If the certificate authority has a defined name, it must be specified._

    {
      type: 'hlfv1',
      name: 'hlfv1org1',
      orderers: [
        'grpc://localhost:7050'
      ],
      ca: {
        url: 'http://localhost:7054',
        name: 'ca.org1.example.com'
      },
      peers: [
        {
          requestURL: 'grpc://localhost:7051',
          eventURL: 'grpc://localhost:7053'
        },
        ],
      channel: 'composerchannel',
      mspID: 'Org1MSP',
      timeout: '300',
    };
```

- `name` 是用于引用连接配置文件的名称，是必需的。

- `type`定义你将连接的Hyperledger Fabric的版本。连接到Hyperledger Fabric v1.0应是`hlfv1`。

- `ca`定义要连接的Hyperledger Fabric证书颁发机构的URL。如果你的证书颁发机构需要一个名称，则必须将其定义为`ca`以上第一个Hyperledger Fabric v1.0示例中所示的属性。

- `trustedRoots`和`verify`证书颁发机构的选项在这里描述<https://fabric-sdk-node.github.io/global.html#TLSOptions>

- `orderers`是描述要与之通信的排序节点的对象数组。在`orderers`中，你必须定义每个排序节点的`url`。如果你通过TLS进行连接，则连接配置文件中的所有`url`属性都必须以`grpcs://`开头，并且还必须在`cert`属性中包含正确的TLS证书。

- `peers`是描述要与之通信的peer的对象数组。每个人都`peer`必须有一个定义`requestURL`和定义`eventURL`。如果你使用TLS进行连接，则每个`peer`都必须在`cert`属性中具有正确的TLS证书。

- `hostnameOverride` 仅在测试环境中使用，当服务器证书的主机名与运行服务器进程的实际主机端点不匹配时，应用程序可以通过将此属性设置为服务器证书主机名的值来解决客户端TLS验证失败的问题。

- 每个`cert`属性的实例都应该是包含PEM格式的TLS证书字符串。可以在每个`cert`属性中放置多个证书。
  ```
  -----BEGIN CERTIFICATE----- ... -----END CERTIFICATE-----
  ```

- `mspid`是您的组织的成员服务提供商ID。它与你将用于与业务网络进行交互的登记ID相关联。

- `timeout`是一个可选属性，用于控制向peer和orderer发出的每个请求的超时时间。请注意，有些命令可能会发出几个连续的请求，超时将分别应用于每个请求。

- `globalCert`定义了在没有指定`cert`属性的情况下，用于所有peer和orderer的TLS证书。如果指定了一个`cert`属性，那么它仅覆盖相应peer或orderer的`globalCert`属性。

- `maxSendSize`是一个可选属性，它定义了发送给orderers和peer的出站grpc消息的大小限制。该值以兆字节定义。如果没有设置，grpc会设置一个默认值。设置此属性为`-1`会导致无大小限制。

- `maxRecvSize`是一个可选属性，用于定义从订购者和同伴接收到的入站grpc消息的大小限制。该值以兆字节定义。如果没有设置，grpc会设置一个默认值。设置此属性为`-1`会导致无大小限制。
