# Hyperledger Composer网络日志级别

`composer network loglevel`命令用于返回或定义Composer运行时的日志级别。如果`newlevel`指定了选项，则会将当前级别更改为指定的值。如果`newlevel`未指定，则此命令将返回当前的日志记录级别。
```bash
composer network loglevel -c admin@tutorial-network
```

### 选项
```
Options:
  --help          Show help  [boolean]
  -v, --version   Show version number  [boolean]
  --newlevel, -l  the new logging level  [choices: "INFO", "WARNING", "ERROR", "DEBUG"]
  --card, -c      The cardname to use to change the log level the network  [string]
```
