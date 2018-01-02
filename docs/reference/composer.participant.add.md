# Composer参与者添加

`composer participant add`命令将参与者的新实例添加到参与者库中。查看“ [添加参与者](../managing_participant-add.md) ”任务，了解使用此命令或API的演练。

`data`选项必须包含一个代表要添加的参与者的序列化JSON串，并且必须用单引号包裹。

## 句法

```bash
composer participant add
composer participant add [options]

Participant options
  --card, -c  The cardname to use to add the participant  [string] [required]
  --data, -d  Serialized participant JSON object as a string  [string] [required]

Options:
  --help         Show help  [boolean]
  -v, --version  Show version number  [boolean]
```

## 选项

`--card, -c` 业务网络卡片，定义要使用的业务网络和身份。例如：`admin@tutorial-network`

`--data, -d` 代表要添加到参与者库的参与者的序列化JSON串。数据必须根据参与者模型有效。

例： `'{"$class":"net.biz.digitalPropertyNetwork.Person","personId":"mae@biznet.org","firstName":"Mae","lastName":"Smith"}'`
