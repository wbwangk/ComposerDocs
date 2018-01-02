# Composer身份绑定

`composer identity bind`命令将现有身份绑定到参与者库中的参与者。查看任务[将现有身份绑定到参与者](../managing/identity-bind.md) 以便体会此命令或API。

## 句法

```bash
composer identity bind
composer identity bind [options]

Options:
  --help                 Show help  [boolean]
  -v, --version          Show version number  [boolean]
  --card, -c             Name of the network card to use  [string] [required]
  --participantId, -a    The particpant to issue the new identity to  [string] [required]
  --certificateFile, -c  File containing the certificate  [string] [required]
```

## 选项

`--card, -c`

要使用的业务网络卡片的名称。例：`admin@sample-network`

`--certificateFile, -c`

以PEM格式包含现有身份证书的文件的路径。
例：`/tmp/cert.pem`

`--participantId, -a`

身份应该颁发给的参与者的完全限定标识符。
例：`resource:net.biz.digitalPropertyNetwork.Person#lenny@biznet.org`
