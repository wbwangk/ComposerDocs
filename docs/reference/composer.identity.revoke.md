# Composer身份撤销

`composer identity revoke`命令撤销参与者库中参与者的现有身份。请参阅[从参与者撤销身份](../managing_identity-revoke.md)的任务以体会此命令或API。

## 句法
```bash
composer identity revoke
composer identity revoke [options]

Options:
  --help                      Show help  [boolean]
  -v, --version               Show version number  [boolean]
  --card, -c                  Name of the network card to use  [string] [required]
  --identityId, -u, --userId  The unique identifier of the identity to revoke  [string] [required]
```

## 选项

`--card, -c` 用于撤销指定身份的业务网络卡片。例：`admin@tutorial-network`

`--identityId, -u`

应该被撤销的现有身份的唯一标识符。
例：`f1c5b9fe136d7f2d31b927e0dcb745499aa039b201f83fe34e243f36e1984862`
