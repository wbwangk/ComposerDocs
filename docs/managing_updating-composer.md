# 更新Hyperledger Composer运行时

部署Hyperledger Composer后，您可能希望升级到新版本。要更新已安装的Hyperledger Composer版本，必须卸载客户端、管理员和运行时CLI组件，并使用npm重新安装它们。

## 过程

1.使用以下命令卸载Hyperledger Composer组件：
```bash
   npm uninstall -g composer-cli
   npm uninstall -g composer-rest-server
   npm uninstall -g generator-hyperledger-composer
```

2.使用以下命令安装最新版本的Hyperledger Composer组件：
```bash
   npm install -g composer-cli
   npm install -g composer-rest-server
   npm install -g generator-hyperledger-composer
```

## 接下来是什么？

- [定义一个商业网络](business-network_bnd-create.md)
- [建模语言](reference_cto_language.md)
- [管理你的解决方案](managing_managingindex.md)
