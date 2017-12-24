# 从REST服务器发布事件

REST服务器可以配置为订阅从已部署的业务网络发出的事件，并发布这些业务事件供客户端应用程序使用。目前，REST服务器支持通过WebSockets向客户端应用程序发布事件。

客户端应用程序可以使用WebSocket客户端来订阅由REST服务器发布的业务事件。WebSocket客户端可用于所有主要编程语言和应用程序类型，例如客户端Web用户界面、后端服务器进程、移动应用程序和集成工具。

## 启用WebSockets

你可以在命令行上使用`-w`参数启用WebSockets ：
```bash
composer-rest-server -c alice1@my-network -w
```

或者，你可以使用`COMPOSER_WEBSOCKETS`环境变量启用WebSockets ：
```bash
export COMPOSER_WEBSOCKETS=true
composer-rest-server -c alice1@my-network
```

成功启用WebSocket后，你将能够将WebSocket客户端连接到REST服务器输出中显示的基本URL：
```
Web server listening at: http://localhost:3000
Browse your REST API at http://localhost:3000/explorer
```

在这个例子中，使用的基本URL是`http://localhost:3000`。你必须改变协议从`http`改成`ws`，来将这个转换为WebSocket的URL。在这个例子中，使用的WebSocket URL是`ws://localhost:3000`。

## 测试WebSocket已启用

你可以通过使用WebSocket客户端来订阅事件来测试已经启用的WebSocket。开源命令行应用程序`wscat`可以用于此目的。

要安装`wscat`，可以使用`npm`。如果你没有正确的权限来全局安装`npm`模块，则可能需要使用sudo或root身份运行此命令：
```bash
npm install -g wscat
```

然后你可以使用`wscat`来连接和订阅由REST服务器发布的业务事件。收到的任何业务事件将被打印到控制台：
```bash
$ wscat -c ws://localhost:3000
connected (press CTRL+C to quit)
< {"$class":"org.acme.sample.SampleEvent","asset":"resource:org.acme.sample.SampleAsset#assetId:1","oldValue":"","newValue":"hello world","eventId":"a80d220b-09db-4812-b04b-d5d03b663671#0","timestamp":"2017-08-23T12:47:17.685Z"}
>
```
