# Composer交易提交

`composer transaction submit`命令将交易提交给业务网络。

## 句法
```bash
composer transaction submit
composer transaction submit [options]

Options:
  --help                       Show help  [boolean]
  -v, --version                Show version number  [boolean]
  -c, --card                   The name of the business network card to use [string] [required]
  --data, -d                   Transactions JSON object as a string  [string] [required]
```

## 选项

`--card, -c` 要使用的业务网络卡片的名称。业务网络卡片用于确定连接和业务网络的详细信息。例：`admin@tutorial-network`

`--data, -d`

要发送到业务网络的事务的序列化JSON陈述。数据必须符合交易的模型定义。
例：`{"$class":"net.biz.digitalPropertyNetwork.RegisterPropertyForSale","transactionId":"TRANSACTION_001","seller":"mae@biznet.org","title":"TITLE_001"}`

## 示例命令

此命令使用用户身份`maeid1`（用户密码`Xurw3yU9zI0l`）提交一个交易到连接配置文件`defaultProfile`连接到的业务网络`digitalproperty-network`中。提交的交易是`'{"$class":"net.biz.digitalPropertyNetwork.RegisterPropertyForSale","transactionId":"TRANSACTION_001","seller":"mae@biznet.org","title":"TITLE_001"}'`。

这是整个命令：
```bash
composer transaction submit -p defaultProfile -n digitalproperty-network -i maeid1 -s Xurw3yU9zI0l -d '{"$class":"net.biz.digitalPropertyNetwork.RegisterPropertyForSale","transactionId":"TRANSACTION_001","seller":"mae@biznet.org","title":"TITLE_001"}'
```
