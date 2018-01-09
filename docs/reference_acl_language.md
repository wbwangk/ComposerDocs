# Hyperledger Composer访问控制语言

Hyperledger Composer包含一个访问控制语言（ACL），提供对领域模型元素的声明式访问控制。通过定义ACL规则，你可以确定允许哪些用户/角色在业务网络的领域模型中创建、读取、更新或删除元素。

## 网络访问控制

Hyperledger Composer区分业务网络（业务访问控制）内的资源的访问控制和网络管理变更（网络访问控制）的访问控制。业务访问控制和网络访问控制都在业务网络的访问控制文件（`.acl`）中定义。

网络访问控制使用系统命名空间，由业务网络中的所有资源隐式扩展; 并授予或拒绝访问下面定义的指定操作，旨在对某些网络级操作进行更细微的访问控制。

### 网络访问控制允许或不允许什么？

网络访问控制会影响以下CLI命令：

#### Composer 网络

**Composer网络部署**

对库和网络的CREATE操作

**Composer 网络下载**

对库和网络的READ操作

**Composer 网络清单**

对库和网络的READ操作

**Composer 网络日志级别**

对网络的UPDATE操作。

**Composer 网络ping**

库和网络上的READ操作。

**Composer 网络取消部署**

库和网络上的DELETE操作。

**Composer 网络更新**

对库的UPDATE或CREATE操作，或对网络上的UPDATE操作。

#### Composer 的身份

**Composer 身份导入**

身份库的UPDATE操作或身份的CREATE操作。

**Composer 的身份发放**

身份库的UPDATE操作或身份的CREATE操作。

**Composer 身份撤销**

身份库的UPDATE操作或身份的DELETE操作。

#### Composer 参与者

**Composer 参与者添加**

参与者的CREATE操作或参与者库的UPDATE操作。

### 授予网络访问控制权

网络访问使用系统命名空间来授予。系统命名空间对于网络访问始终是`org.hyperledger.composer.system.Network`，对于所有访问始终是`org.hyperledger.composer.system`。以下访问控制规则赋予**networkControl**参与者使用网络命令的所有操作的权限。
```
rule networkControlPermission {
  description:  "networkControl can access network commands"
  participant: "org.acme.vehicle.auction.networkControl"
  operation: ALL
  resource: "org.hyperledger.composer.system.Network"
  action: ALLOW  
}
```

以下访问控制规则将使所有参与者都能访问业务网络中的所有操作和命令，包括网络访问和业务访问。
```
rule AllAccess {
  description: "AllAccess - grant everything to everybody"
  participant: "org.hyperledger.composer.system.Participant"
  operation: ALL
  resource: "org.hyperledger.composer.system.**"
  action: ALLOW
}
```

## 访问控制规则的评估

业务网络的访问控制由一组有序的ACL规则来定义。规则按顺序进行评估，条件匹配的第一条规则决定访问是被授予还是被拒绝。如果没有规则匹配，则访问被**拒绝**。

ACL规则是在业务网络的根目录中一个叫`permissions.acl`的文件中定义的。如果该文件从业务网络中缺失，则**允许**所有访问。

## 访问控制规则语法

有两种类型的ACL规则：简单ACL规则和条件ACL规则。简单的规则根据参与者类型或参与者实例对命令空间、资产、资产或资产属性的访问进行控制。

例如，下面的规则指出，`org.example.SampleParticipant`类型的所有实例都可以对`org.example.SampleAsset`的所有实例执行所有操作。
```
rule SimpleRule {
    description: "Description of the ACL rule"
    participant: "org.example.SampleParticipant"
    operation: ALL
    resource: "org.example.SampleAsset"
    action: ALLOW
}
```

条件ACL规则为参与者和被访问的资源引入变量绑定，以及一个布尔型JavaScript表达式，如果为true则允许参与者访问资源，否则拒绝访问。

例如，下面的规则指出，所有`org.example.SampleParticipant`类型的实例都可以执行所有`org.example.SampleAsset`示例的所有操作，只要那个参与者是这个资产的所有者。
```
rule SampleConditionalRule {
    description: "Description of the ACL rule"
    participant(m): "org.example.SampleParticipant"
    operation: ALL
    resource(v): "org.example.SampleAsset"
    condition: (v.owner.getIdentifier() == m.getIdentifier())
    action: ALLOW
}
```

条件ACL规则也可以指定一个可选的交易子句。当指定交易子句时，ACL规则只允许发起交易的参与者访问资源，并且该交易是指定的类型。

例如，下面的规则指出，任何`org.example.SampleParticipant`类型的实例都可以访问`org.example.SampleAsset`的所有示例执行所有操作，如果那个参与者是这个资产的所有者，并且参与者提交`org.example.SampleTransaction`类型的交易以执行操作。
```
rule SampleConditionalRuleWithTransaction {
    description: "Description of the ACL rule"
    participant(m): "org.example.SampleParticipant"
    operation: READ, CREATE, UPDATE
    resource(v): "org.example.SampleAsset"
    transaction(tx): "org.example.SampleTransaction"
    condition: (v.owner.getIdentifier() == m.getIdentifier())
    action: ALLOW
}
```

可以定义多个ACL规则，从概念上定义决策表。决策树的行为定义了访问控制决策（ALLOW或DENY）。如果决策表无法匹配，则默认访问被拒绝。

**资源**定义了ACL规则适用的东西。这可以是一个类、一个命名空间中的所有类、或者一个命名空间下的所有类。它也可以是一个类的实例。

资源示例：

- 命名空间：`org.example.*`

- 命名空间（递归）：`org.example.**`

- 命名空间中的类：`org.example.Car`

- 类的实例：`org.example.Car#ABC123`

**操作**标识规则管理的操作。支持四种操作：CREATE、READ、UPDATE和DELETE。你可以使用ALL指定规则管理所有支持的操作。或者，可以使用逗号分隔列表来指定规则管理一组支持的操作。

**参与者**定义已提交交易进行处理的个人或实体。如果指定了参与者，他们必须存在于参与者库中。PARTICIPANT可以选择性地绑定到一个变量，用于PREDICATE。特殊值“ANY”可以用来表示参与者类型检查不被强制执行。

**交易**定义参与者为了对指定资源执行指定操作而必须提交的交易。如果指定了此子句，并且参与者未提交此类型的交易（例如，他们正在使用CRUD API），则ACL规则不允许访问。

**条件**是绑定变量上的布尔型JavaScript表达式。任何带有`if(...)`的JavaScript表达式都是合法，可以在这里使用。用于ACL规则条件的JavaScript表达式可以引用脚本文件中的JavaScript工具函数。这允许用户轻松实现复杂的访问控制逻辑，并在多个ACL规则中重复使用相同的访问控制逻辑函数。

**操作**标识规则的操作。它必须是两个之一：ALLOW、 DENY。

## 例子

ACL规则示例（按评估顺序）：
```
rule R1 {
    description: "Fred can DELETE the car ABC123"
    participant: "org.example.Driver#Fred"
    operation: DELETE
    resource: "org.example.Car#ABC123"
    action: ALLOW
}

rule R2 {
    description: "regulator with ID Bill can not update a Car if they own it"
    participant(r): "org.example.Regulator#Bill"
    operation: UPDATE
    resource(c): "org.example.Car"
    condition: (c.owner == r)
    action: DENY
}

rule R3 {
    description: "regulators can perform all operations on Cars"
    participant: "org.example.Regulator"
    operation: ALL
    resource: "org.example.Car"
    action: ALLOW
}

rule R4 {
    description: "Everyone can read all resources in the org.example namespace"
    participant: "ANY"
    operation: READ
    resource: "org.example.*"
    action: ALLOW
}

rule R5 {
    description: "Everyone can read all resources under the org.example namespace"
    participant: "ANY"
    operation: READ
    resource: "org.example.**"
    action: ALLOW
}
```

规则从顶部（最具体）到底部（最不具体）进行评估。一旦参与者、操作和资源匹配一个规则，那么后续规则就不会被评估。

这种排序使决策表更快地扫描人和计算机。如果没有ACL规则触发，那么访问控制决定是DENY（拒绝）。
