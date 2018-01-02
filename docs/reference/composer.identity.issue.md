# Composer身份颁发

`composer identity issue`命令向参与者库（关联到某个业务网络）中的一个参与者颁发一个新的身份。通过查看任务[为参与者颁发新身份](../managing_identity-issue.md)，漫游此命令或API。

## 句法
```bash
composer identity issue

Business Network Cards
  --card, -c  Name of the network card to use for issuing  [string]
  --file, -f  The card file name for the new identity  [string]

Identity Options
  --newUserId, -u      The user ID for the new identity  [string]
  --participantId, -a  The participant to issue the new identity to  [string] [required]
  --issuer, -x         If the new identity should be able to issue other new identities  [boolean]

Options:
  --help             Show help  [boolean]
  -v, --version      Show version number  [boolean]
  --option, -o       Options that are specific specific to connection. Multiple options are specified by repeating this option  [string]
  --optionsFile, -O  A file containing options that are specific to connection  [string]
```

## 选项

`--card, -c`

用于颁发身份的业务网络卡片的名称。例：`admin@tutorial-network`

`--file, -f`

要创建的卡片文件的文件名称，注意这不是要创建的身份的名称。例：`DanSelman`

`--newUserId, -u` 新身份的用户ID，这是新身份的名称。例：`Dan`

`--participantId, -a` 身份应该颁发给的参与者的完全限定标识符（以URI形式）。
例：`resource:net.biz.tutorial-network.Person#DanSelman@biznet.org`

`--issuer, -x` 新身份是否能够颁发其他新身份。
