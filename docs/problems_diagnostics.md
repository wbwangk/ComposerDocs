# 诊断问题

Composer默认使用Winston日志记录模块，并使用Config模块查找任何配置信息。如果没有找到，那么将使用一组默认值。

如果没有设置配置文件，配置模块会写出警告。例如。`WARNING: No configurations found in configuration directory`。如果您对默认值感到满意，并且不希望在应用程序中使用配置，则可以使用环境变量来抑制这种情况。[在这里](https://github.com/lorenwest/node-config/wiki/Environment-Variables#suppress_no_config_warning)查看更多信息。

# 诊断问题

如果应用程序出现问题，应该如何处理诊断？

我们来看一下`digitalproperty-app`示例，并用它来解释如何从框架中获取诊断信息。

> 请注意：这是一个框架 - 所以你的应用程序将需要它自己的日志框架。另外，您的应用程序也可以用配置信息来控制Hyperledger Composer自己的日志记录。

有两个容器与日志相关：

- 运行应用程序的容器

- 执行交易函数的链码容器。

## 应用

在内部，默认情况下，Hyperledger Composer使用[Winston](https://github.com/winstonjs/winston) node.js日志包，初始级别为日志点和目标。

## 默认配置

该框架将记录这些级别的信息。

- 错误

- 警告

- 信息

- 详细

- 调试

“Silly”级别没有被使用。

## 信息的位置

默认情况下，有两个数据位置：

- 一个文本文件，位于`${CurrentWorkingDir}/logs/trace_<processid>.trc`

- 另一个是`stdout`。默认情况下，`stdout`只显示任何以“错误”级别记录的数据，文件将显示“信息”或以上（即信息，警告和错误）日志数据。

## 控制生成什么

配置模块用于定位信息以控制如何生成日志。

例如
```json
{
  "gettingstarted": {
       "participantId" : "WebAppAdmin",
       "participantPwd" :"DJY27pEnl16d",
       "businessNetworkIdentifier" : "digitalproperty-network",
       "connectionProfile" :"defaultProfile"
    },
    "ComposerConfig": {
       "debug": {
         "logger": "default",
         "config": {
           "console": {
             "enabledLevel": "info",
             "alwaysLevel": "none"

           },
           "file": {
             "filename": "./trace_PID.log",
             "enabledLevel": "silly",
             "alwaysLevel": "info"
           }
         }
       }
      }

}
```

第一部分针对Getting Started应用程序，第二个`ComposerConfig`部分针对Hyperledger Composer。

- `logger`用于指定实际记录日志的模块。默认是暗示这是winston框架

- `config`传递给日志记录器来控制它做什么。所以这部分是特定于使用的记录器。

## 启用更多信息

调试node.js应用程序的标准方法是使用`DEBUG`环境变量。所以呢
```
DEBUG=composer:* node myApplication.js
```

将启用更多的跟踪 - 这将使用`enabledLevel`上面配置中标记的级别。

# 如何找出哪个链码容器有已部署网络？

每个业务网络都部署到自己的链码容器中。在使用事务函数说错误的情况下，查看来自该容器的日志并查看发生了什么可能会有所帮助。

要确定哪个Docker容器具有部署的网络，请运行以下命令：
```
docker ps
```

Docker容器名称应包含已部署网络的名称。
