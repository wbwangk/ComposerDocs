# 创建一个业务网络定义

业务网络定义具有以下布局：
```
models/ (optional)
lib/
permissions.acl (optional)
package.json
README.md (optional)
```

创建新的业务网络定义的最简单的方法是`git clone`范例，或者使用Hyperledger Composer Yeoman生成器来创建骨架业务网络。

## README.md

使用Markdown标记语言描述业务网络的意图。

## Package.json

业务网络定义有一个名称（仅限于基本的ASCII字母数字字符和`-`）、一个可读的描述和一个版本号。网络的版本号应采用Major.Minor.Micro形式 ，增加版本号时应使用[Semantic Versioning](http://semver.org/)原则。

网络的标识符由其名称、`-`字符和版本号组成。一个有效的标识符（和例子）类似于`mybusinessnetwork-1.0.3`。

业务网络定义的元数据从`package.json`中读取，这意味着业务网络定义也是有效的`npm`包。

## 模型

业务网络定义的领域模型集定义了用于网络内的有效类型，和用于与外部系统（例如，向网络提交交易的系统）集成的有效类型。

领域模型可以打包在业务网络定义中（通常存储在`models`目录下），也可以在`package.json`中声明为外部依赖。如果您想跨业务网络定义共享模型，则可以通过npm依赖引用模型。

## 脚本

业务网络定义的脚本通常存储在`lib`目录下，并打包在业务网络定义中。这些脚本是用ES 5 JavaScript编写的，并引用在业务网络的域模型中定义的类型。

## Permissions.acl

表示的业务网络的权限表示在一个可选`permissions.acl`文件中。

# 克隆一个示例业务网络定义

示例业务网络定义存储在GitHub上`https://github.com/hyperledger/composer-sample-networks`。你可以通过`git clone`这个存储库来拉取所有的示例网络。每个示例网络都存储在`packages`目录下。

# 生成业务网络定义

## 生成

1.`yo hyperledger-composer`

```
Welcome to the Hyperledger Composer Skeleton Application Generator
? Please select the type of Application: (Use arrow keys)
❯ CLI Application
  Angular 2 Application
  Skeleton Business Network
```
并选择 `Skeleton Business Netork`

2.回答所有的问题

```
? Please select the type of Application: Skeleton Business Network
You can run this generator using: 'yo hyperledger-composer:businessnetwork'
Welcome to the business network skeleton generator
? Do you only want to generate a model? Yes
? What is the business network's name? mynetwork
? What is the business network's namespace? org.example
? Describe the business network This is my test network
? Who is the author? Dan Selman
? Which license do you want to use? Apache-2
   create index.js
   create package.json
   create README.md
   create models/org.example.cto
   create .eslintrc.yml
```

这产生了骨架业务网络与`asset`、`participant`和`transaction`定义，以及一个`mocha`单元测试。

还包括一个“最佳实践”eslint配置文件，它定义了JS代码的示例linting 规则。

## 参考

- [**建模语言**](reference_cto_language.md)

- [**访问控制语言**](reference_acl_language.md)

- [**交易处理器函数**](reference_js_scripts.md)

