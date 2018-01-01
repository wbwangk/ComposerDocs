# Hyperledger Composer网络取消部署

`composer network undeploy`命令将**永久禁用一个业务网络**。一旦一个业务网络被取消部署，它就不能被重新部署。

**请注意**：当对一个在Hyperledger Fabric v1.0上运行的业务网络使用`undeploy`命令时，业务网络将保持运行，但将无响应。当`undeploy`命令发出后，业务网络**不能被重新部署或更新**。这是因为业务网络已经部署，但已被设置为无响应。
```bash
composer network undeploy -c <card-name>
```

请注意，**取消部署后，业务网络定义将不能再使用**，但与业务网络定义相关联的docker容器仍在运行。如果不再需要，业务网络定义的docker容器必须明确地停止和删除。

### 选项
```bash
Options:
  --help         Show help  [boolean]
  -v, --version  Show version number  [boolean]
  --card, -c     The business network card to use to undeploy the network  [string]
```

## 输出示例
```bash
composer network undeploy -c admin@tutorial-network
Undeploying business network definition. This may take some seconds...
Command completed successfully.
```
