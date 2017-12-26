# 在本地安装和运行Hyperledger Composer Playground

本教程将带你了解如何在本地计算机上安装和运行Hyperledger Composer Playground。它还创建了Hyperledger Fabric v1.0的一个实例。

Hyperledger Composer Playground还可以在“仅浏览器”模式下使用，不需要运行Hyperledger Fabric实例。在此模式下使用时，Hyperledger Composer Playground的所有功能都可用，但所有数据（业务网络、资产、参与者和事务）都会保留在浏览器本地存储中。

## 在你开始之前

为了安装Hyperledger Composer Playground，你需要安装以下软件：

- 操作系统：Ubuntu Linux 14.04 / 16.04 LTS（均为64位）或Mac OS 10.12
- Docker引擎17.03或更高
- Docker Compose 1.8或更高版本

*请注意：*如果你以前曾经在本地使用Hyperledger Composer Playground或Hyperledger Fabric并希望清除所有内容并重新开始，则以下命令将删除所有正在运行的容器并删除所有下载的图像，（如果在你的机器上要使用其他Docker映像请小心）：
```bash
docker ps -aq | xargs docker rm -f
docker images -aq | xargs docker rmi -f
```

要运行Hyperledger Composer和Hyperledger Fabric，我们建议你至少拥有4Gb的内存。

## 在本地创建容器并安装Playground

1. 选择要安装的目录，然后运行以下命令下载并启动Hyperledger Fabric实例和Hyperledger Composer Playground：
   ```bash
   curl -sSL https://hyperledger.github.io/composer/install-hlfv1.sh | bash
   ```

2. 点击以下链接访问你当地的Hyperledger Composer Playground：[http://localhost:8080](http://localhost:8080/)。*请注意*：在本地运行Playground时，不支持私密浏览。

## 下一步是什么？

- 寻找使用Playground的教程，请尝试[我们的Playground教程](https://hyperledger.github.io/composer/tutorials/playground-tutorial.html)。
