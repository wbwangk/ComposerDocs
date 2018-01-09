# 测试业务网络

Hyperledger Composer支持三种类型的测试：交互式测试、自动化单元测试和自动系统测试。三者都有不同的用途，对于确保区块链项目的成功至关重要。

在部署了业务网络定义之后，通常运行一个互动的“冒烟测试”以确保部署成功。为了运行这样的冒烟测试，`composer`CLI暴露了几个命令。

另一方面，你可以使用Docker Compose和Mocha/Chai编写完整的系统测试，这些测试启动运行时，部署业务网络定义，然后以编程方式创建资产、提交交易并查看资产库状态。

在fall单元测试之间。单元测试着重于确保在处理交易时对世界状态进行正确的更改。

单元测试和系统测试的执行可以使用CI/CD构建流水线，例如Jenkins、Travis CI或Circle CI或其他。

## 交互式测试

你可以使用Playground以交互方式测试创建参与者、资产和提交交易。

## 从命令行进行测试

命令行可用于检查运行时的状态并提交交易。使用`composer network list`命令查看资产和参与者库的状态。使用`composer transaction submit`命令提交交易。

## 创建单元测试

交易处理器函数中的业务逻辑应该具有单元测试，理想情况下具有100％的代码覆盖率。这将确保你在业务逻辑中没有错别字或逻辑错误。

你可以使用标准的JavaScript测试库（如Mocha，Chai，Sinon和Istanbul）来单元测试你的交易处理器函数中的逻辑。

对于单元测试，`embedded`运行时非常有用，因为它可以让你在一个模拟的Node.js区块链环境中快速测试业务逻辑，而不必启动一个Hyperledger Fabric。

请参考示例网络中的单元测试示例。例如：https://github.com/hyperledger/composer-sample-networks/blob/master/packages/bond-network/test/Bond.js

## 参考

- [**Composer CLI命令**](reference_commands.md)

- [**Mocha**](https://mochajs.org/)

- [**Chai**](http://chaijs.com/)

- [**Sinon**](http://sinonjs.org/)

- [**Istambul**](https://istanbul.js.org/)

