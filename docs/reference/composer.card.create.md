# Composer卡片创建

从各个组件创建一个业务网络卡片。创建业务网络卡片时，你将需要一个`enrollSecret`或一对 `certificate`和`privateKey`。
```bash
composer card create --file conga.card --businessNetworkName penguin-network --connectionProfileFile connection.json --user conga --enrollSecret supersecret
```

##句 法
```
Card options
  --file, -f                   File name of the card archive to be created  [string]
  --businessNetworkName, -n    The business network name  [string]
  --connectionProfileFile, -p  Filename of the connection profile json file  [string] [required]
  --user, -u                   The name of the identity for the card  [string] [required]
  --enrollSecret, -s           The enrollment secret of the user  [string]
  --certificate, -c            File containing the user's certificate.  [string]
  --privateKey, -k             File containing the user's private key  [string]
  --role, -r                   The role for this card can, specify as many as needed  [choices: "PeerAdmin", "ChannelAdmin"]

Options:
  --help         Show help  [boolean]
  -v, --version  Show version number  [boolean]
```
