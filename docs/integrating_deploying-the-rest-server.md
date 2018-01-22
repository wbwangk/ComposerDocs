# 为业务网络部署REST服务器

在生产环境（例如使用Docker Swarm或Kubernetes）中部署Hyperledger Composer REST服务器时，应将REST服务器配置为高度可用。这意味着你必须部署多个REST服务器实例，并且应该配置这些实例以共享数据。例如，应共享连接配置文件，区块链身份和REST API身份认证设置等数据，以便REST API客户端可以向任何实例发出请求，而无需重新进行身份认证。

## 业务网络卡片和业务网络卡片存储

REST服务器使用在启动期间指定的业务网络卡片来连接并发现已部署业务网络中的资产、参与者和交易。这些信息是为了生成REST API所必需的。这个业务网络卡片被称为发现业务网络卡片。默认情况下，发现业务网络卡片也用于处理对REST API的所有请求。但是，REST服务器也可以配置为多用户模式，允许经过身份认证的用户提供自己的业务网络卡片来处理对REST API的请求。

为了使用发现业务网络卡片，必须首先将该业务网络卡片导入到REST服务器可用的业务网络卡片存储中。默认的业务网络卡片存储是一个本地文件系统目录`~/.composer`（其中`~`是当前用户主目录）。REST服务器使用Docker映像时，必须挂载一个卷到默认业务网络卡片存储区，其中应包含导入的发现业务网络卡片。在REST服务器的Docker镜像中，REST服务器使用的业务网络卡片存储位于目录`/home/composer/.composer`（因为Docker镜像中的REST服务器始终在`composer`用户下运行）。

业务网络卡片包含一个连接配置文件，用于描述如何连接到正在运行的已部署业务网络的Hyperledger Fabric网络。请注意，连接配置文件必须在用于REST服务器的Docker镜像中有效，并且主机名必须正确且可由此Docker镜像访问。

## 使用持久数据存储配置REST服务器

有关已认证用户及其钱包（多用户模式启用时包含多个用户的业务网络卡片）的所有信息将通过LoopBack连接器持久化在LoopBack数据源中。默认情况下，REST服务器使用LoopBack“内存”连接器来持久化用户信息，当REST服务器终止时会丢失这些信息。REST服务器应配置一个LoopBack连接器，用于将数据存储在高度可用的数据源（例如数据库）中。

你可以使用任何LoopBack连接器，但是我们建议你使用NoSQL数据库(作为源的)LoopBack连接器。例如，MongoDB或Apache CouchDB。

需要安装LoopBack连接器才能让REST服务器找到并使用它。你可以使用`npm`安装附加LoopBack连接器，例如：
```bash
npm install -g loopback-connector-mongodb
```

最后，你需要为REST服务器提供LoopBack连接器所需的连接信息。这个连接信息应该使用`COMPOSER_DATASOURCES`环境变量来提供。有关可用于配置REST服务器的环境变量的更多信息，请参阅参考文档：[Hyperledger Composer REST服务器](reference_rest-server.md)

## 用附加的Node.js模块扩展REST服务器的Docker镜像

为了将REST服务器部署为具有附加LoopBack连接器和Passport策略的Docker容器，你必须扩展`hyperledger/composer-rest-server`Docker镜像。

以下是一个Dockerfile示例，它将MongoDB的LoopBack连接器和GitHub的Passport策略添加到Docker镜像中：
```dockerfile
FROM hyperledger/composer-rest-server
RUN npm install --production loopback-connector-mongodb passport-github && \
    npm cache clean --force && \
    ln -s node_modules .node_modules
```

你可以通过将Dockerfile放置在一个目录中并使用`docker build`命令来构建此Docker镜像，例如：
```bash
docker build -t myorg/my-composer-rest-server .
```

你可能需要将此Docker镜像发布到Docker镜像库（例如Docker Hub），以便将其用于基于云的Docker部署服务。

## 使用Docker部署持久和安全的REST服务器

以下的例子将演示如何使用Docker部署REST服务器。部署的REST服务器将使用MongoDB保存数据，并使用GitHub身份认证进行保护。

这些例子基于作为开发者教程的一部分部署到Hyperledger Fabric v1.0的业务网络，可能需要调整你的配置，例如，如果Docker网络名称不匹配。

1.通过运行以下`composer network ping`命令，确保你的业务网络的有效业务网络卡片位于本地业务网络卡片存储中。这个例子在业务网络`my-network`上，使用了用户`admin`的业务网络卡片：
```bash
   composer network ping -c admin@my-network
```

   请注意，在继续之前，你**必须**使用`composer network ping`命令来测试与业务网络的连接。如果业务网络卡片只包含了用户ID和登记密码，则`composer network ping`命令会触发登记过程，并将证书存储在业务网络卡片中。以Docker镜像方式运行REST服务器时，**不**建议使用只有一个用户ID和登记密码的业务网络卡片。

2.启动一个名为`mongo`的MongoDB Docker实例。这个MongoDB实例被REST服务器用来保存被认证用户和他们钱包（当多个用户模式被启用时会包含多个用户业务网络卡片）的所有信息。
```bash
   docker run -d --name mongo --network composer_default -p 27017:27017 mongo
```

   请注意，MongoDB实例已连接到名为`composer_default`的Docker网络。这意味着在Docker网络`composer_default`中可以使用主机名`mongo`来访问MongoDB实例。在随后的步骤我们将使用主机名`mongo`来配置REST服务器。根据你的Docker网络配置，你可能需要指定不同的Docker网络名称。MongoDB端口`27017`也使用端口`27017`暴露在主机网络上，因此如果需要，你也可以使用其他MongoDB客户端应用与此MongoDB实例进行交互。

3.通过添加用于MongoDB的LoopBack连接器和用于GitHub身份认证的Passport策略，来扩展REST服务器的Docker镜像。在本地文件系统上创建一个新的空目录，并在新目录中创建一个名为`Dockerfile`新文件，内容如下：

```Dockerfile
   FROM hyperledger/composer-rest-server
   RUN npm install --production loopback-connector-mongodb passport-github && \
       npm cache clean --force && \
       ln -s node_modules .node_modules
```

   在包含新建的`Dockerfile`文件的目录中运行以下的`docker build`命令来构建扩展的Docker镜像：
```bash
   docker build -t myorg/my-composer-rest-server .
```

   如果此命令成功完成，则会创建一个新的名为`myorg/my-composer-rest-server`的Docker镜像，并将其存储在系统的本地Docker镜像库中。如果你想在其他主机上使用这个Docker镜像，你可能需要将Docker镜像推送到Docker镜像库，如Docker Hub。

4.REST服务器的Docker映像是使用环境变量而不是命令行选项配置的。创建一个名为`envvars.txt`的新文件，为REST服务器存储环境变量，其中包含以下内容：
```bash
   COMPOSER_CARD=admin@my-network
   COMPOSER_NAMESPACES=never
   COMPOSER_AUTHENTICATION=true
   COMPOSER_MULTIUSER=true
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
   COMPOSER_DATASOURCES='{
       "db": {
           "name": "db",
           "connector": "mongodb",
           "host": "mongo"
       }
   }'
```

   请注意，发现业务网络卡片的名称`admin@my-network`已被设置为`COMPOSER_CARD`环境变量的值。我们已经通过环境变量`never`的值来禁用生成的REST API中的命名空间`COMPOSER_NAMESPACES`。我们通过设置环境变量`COMPOSER_AUTHENTICATION`的值为`true`来启用REST API客户端身份认证，并通过设置环境变量`COMPOSER_MULTIUSER`的值为`true`来启用多用户模式。

   我们通过环境变量`COMPOSER_PROVIDERS`来设置REST服务器使用GitHub的Passport策略以使用GitHub身份认证。请注意，你必须将`REPLACE_WITH_CLIENT_ID`和`REPLACE_WITH_CLIENT_SECRET`替换为GitHub中的正确值，才能使这个配置成功运行。

   我们通过在`COMPOSER_DATASOURCES`环境变量中配置MongoDB的LoopBack连接器来让REST服务器使用MongoDB实例。请注意，MongoDB实例的主机名`mongo`被指定名为`db`的LoopBack数据源的`host`属性值。

   通过运行以下命令将环境变量加载到当前shell中：
```bash
   source envvars.txt
```

   如果你打开一个新的shell，例如一个新的终端窗口或标签，那么你必须再次运行相同的`source`命令，把环境变量加载到新的shell中。

   关于配置REST服务器的环境变量的更多信息，请参阅参考文档：[Hyperledger Composer REST服务器](reference_rest-server.md)

5.通过运行以下`docker run`命令，为步骤3中创建的REST服务器的扩展Docker镜像启动新实例：

```bash
   docker run \
       -d \
       -e COMPOSER_CARD=${COMPOSER_CARD} \
       -e COMPOSER_NAMESPACES=${COMPOSER_NAMESPACES} \
       -e COMPOSER_AUTHENTICATION=${COMPOSER_AUTHENTICATION} \
       -e COMPOSER_MULTIUSER=${COMPOSER_MULTIUSER} \
       -e COMPOSER_PROVIDERS="${COMPOSER_PROVIDERS}" \
       -e COMPOSER_DATASOURCES="${COMPOSER_DATASOURCES}" \
       -v ~/.composer:/home/composer/.composer \
       --name rest \
       --network composer_default \
       -p 3000:3000 \
       myorg/my-composer-rest-server
```

   请注意，我们通过使用多个`-e`选项传入前面步骤中设置的所有环境变量。如果你需要添加或删除其他环境变量来配置REST服务器，则需要添加或删除相应的`-e`选项。

   我们已经通过指定`-v ~/.composer:/home/composer/.composer`将本地业务网络卡片存储挂载到REST服务器的Docker容器中。这允许REST服务器在使用`COMPOSER_CARD`环境变量尝试加载发现业务网络卡片时，访问和使用本地的业务网络存储卡片。

   我们还指定了Docker网络名称`composer_default`和Docker容器的名称`rest`。这意味在`composer_default`Docker网络上，可以使用主机名`rest`访问REST服务器实例。REST服务器端口`3000`也使用端口`3000`暴露在主机网络上。

   你可以使用`docker logs`命令检查REST服务器是否已成功启动，例如：
```bash
   docker logs -f rest
```

   如果REST服务器已经成功启动，那么你将看到它输出的日志消息类似于`Browse your REST API at http://localhost:3000/explorer`。

现在REST服务器已经成功启动，你可以使用以下URL访问运行在Docker容器内的REST服务器：[http://localhost:3000/explorer/](http://localhost:3000/explorer/)。

## 最后的笔记

在本指南中，你已经看到了如何使用Docker启动REST服务器的单个实例，其中单个实例被配置为使用MongoDB作为持久数据存储。要获得真正高度可用的REST服务器的生产部署，你需要：

- 配置持久数据存储的高可用性实例，例如MongoDB副本集。

- 运行REST服务器Docker镜像的多个实例。通过使用`--name`参数更改Docker容器的名称以及使用参数`-p 3000:3000`更新或删除后续REST服务器实例的主机端口映射。

- 部署负载均衡器（例如Nginx），以便将客户端的REST请求分发到REST服务器的所有实例。

完成这三项任务后，你应能够停止、重新启动或删除任何REST服务器实例（但不是全部！），而不会丢失通过REST访问已部署的业务网络的访问。

