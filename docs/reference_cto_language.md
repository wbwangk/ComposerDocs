# Hyperledger Composer建模语言

Hyperledger Composer包含一个面向对象的建模语言，用于定义业务网络定义的领域模型。

Hyperledger Composer CTO文件由以下元素组成：

1. 一个单一的命名空间。文件中的所有资源声明都隐含在这个命名空间中。  

2. 一组资源定义，包括资产、交易、参与者和事件。  

3. 从其他命名空间导入资源的可选导入声明。  

## 组织和Hyperledger Composer系统命名空间

你的组织命名空间是在你的model（`.cto`）文件的命名空间行中定义的，并且所有创建的资源隐式地都是该命名空间的一部分。

除了定义资产、参与者、事件和交易的新类外，还有一个[系统命名空间](https://github.com/hyperledger/composer/blob/master/packages/composer-common/lib/system/org.hyperledger.composer.system.cto)，其中包含资产、事件、参与者和交易的基本定义。这些基本定义是由所有资产、事件、参与者和交易隐含扩展的抽象类型。

在系统命名空间定义中，资产和参与者没有必需的值。事件和交易由eventId或transactionId和时间戳定义。系统命名空间还包括Registry(库)的定义、历史记录、身份和一些系统交易。

> 如果你定义了一个包含eventId、transactionId或timestamp的事件或交易，则必须删除eventId、transactionId或timestamp属性。

## 资源声明

Hyperledger Composer中的资源包括：

- 资产、参与者、交易和事件。
- 枚举类型。
- 概念。

资产、参与者和交易是类定义。资产、参与者和交易的概念可以认为是类的不同模板。

Hyperledger Composer中的类被称为资源定义，因此一个资产实例具有一个资产定义。

资源定义具有以下属性：

1.由其父文件的命名空间定义的命名空间。`.cto`文件的命名空间隐式应用于其中创建的所有资源。  
2.一个名称（例如`Vehicle`）和一个id字段（例如`vin`）。如果资源是资产或参与者，则名称后跟标识id字段，如果资源是事件或交易，则自动生成id字段。在这个例子中，资产名称是`Vehicle`，而id字段是`vin`。
```
   /**
    * A vehicle asset.
    */
   asset Vehicle identified by vin {
     o String vin
   }
```

3.一个可选超类，是资源定义扩展。该资源将具有超类所定义的所有属性和字段，并从自己的定义中添加附加属性或字段。
```
   /**
    * A car asset. A car is related to a list of parts
    */
   asset Car extends Vehicle {
     o String model
     --> Part[] Parts
   }
```

4.一个可选的“抽象”声明，表示不能创建这种类型。抽象资源可以作为其他类的基础来扩展。抽象类的扩展不会继承抽象类的状态。例如，上面定义的`Vehicle`资产不应该创建，因为应该定义更多的特定资产类来扩展它。
```
   /**
   * An abstract Vehicle asset.
   */
   abstract asset Vehicle identified by vin {
     o String vin
   }
```

5.一组命名的属性。属性必须被命名，并且用原始数据类型定义。属性及其数据由每个资源拥有，例如，`Car`资产具有`vin`和`model`属性，两者都是字符串。  

6.与其他Composer类的一组关联，这些类不属于资源，但可以从资源引用。关联是单向的。  
```
/**
* A Field asset. A Field is related to a list of animals 
*/ 
asset Field identified by fieldId {
  o String fieldId o String name 
  --> Animal[] animals 
}
```

### 枚举类型的声明

枚举类型是用于指定可能有1或N个可能值的类型。下面的例子定义了ProductType枚举，它可能的值有`DAIRY`或`BEEF`或`VEGETABLES`。
```
/**
* An enumerated type
*/
enum ProductType {
o DAIRY
o BEEF
o VEGETABLES
}
```

当创建另一个资源时，例如参与者，可以根据枚举类型定义该资源的属性。
```
participant Farmer identified by farmerId {
    o String farmerId
    o ProductType primaryProduct
```

### 概念

概念是非资产、参与者或交易的抽象类。它们通常被资产、参与者或交易包含。

例如，下面定义了一个抽象概念`Address`，然后个性化成一个`UnitedStatesAddress`。请注意，概念没有`identified by`字段，因为它们不能直接存储在库中或在关联中引用。
```
abstract concept Address {
  o String street
  o String city default ="Winchester"
  o String country default = "UK"
  o Integer[] counts optional
}

concept UnitedStatesAddress extends Address {
  o String zipcode
}
```

### 原始类型

Composer资源是根据以下原始类型定义的：

1. 字符串：一个UTF8编码的字符串。

2. Double：双精度64位数值。

3. 整数：一个32位有符号整数。

4. 长：64位有符号整数。

5. DateTime：与ISO-8601兼容的时间实例，具有可选的时区和UTZ偏移量。

6. Boolean：布尔值，true或false。

### 数组

Composer中的所有类型都可以使用[]符号声明为数组。
```
Integer[] integerArray
```

整数数组存储在一个名为“integerArray”的字段中。而
```
--> Animal[] incoming
```
是一个指向到动物类型的关联数组，存储在一个名为“incoming”的字段中。

### 关联

Composer语言中的关联是由以下组成的元组：

1. 被引用类型的命名空间

2. 被引用类型的类型名称

3. 被引用的实例的标识符(id)

因此，一个关联可能是：org.example.Vehicle＃123456

这是一个到Vehicle类型的关联，它在org.example命名空间声明，具有123456的id。

关联是单向的，删除不会级联，即就是说，消除关联对它指向的东西没有影响。删除被指向的东西不会使关联无效。

关联必须被*解析*才能获取被引用对象的实例。如果对象不存在或关联中的信息无效，则解析行为可能会导致空。

### 字段验证器

字符串字段可能包含一个(可选)正则表达式，用于验证字段的内容。仔细使用字段验证器可以使Composer执行丰富的数据验证，从而减少错误，减少样板代码。

下面的例子声明`Farmer`参与者包含一个`postcode`字段，该字段利用正则表达式确保必须符合英国有效邮政编码。

```
participant Farmer extends Participant {
    o String firstName default="Old"
    o String lastName default="McDonald"
    o String address1
    o String address2
    o String county
    o String postcode regex=/(GIR 0AA)|((([A-Z-[QVf]][0-9][0-9]?)|(([A-Z-[QVf]][A-Z-[IJZ]][0-9][0-9]?)|(([A-Z-[QVf]][0-9][A-HJKPSTUW])|([A-Z-[QVf]][A-Z-[IJZ]][0-9][ABEHMNPRVWfY])))) [0-9][A-Z-[CIKMOV]]{2})/
}
```

Double，Long或Integer字段可以包含一个可选的范围表达式，用于验证字段的内容。

下面的例子声明`Vehicle`资产有一个`year`默认值是2016的Integer字段，并且必须是1990或更高。如果检查不需要，则范围表达式可以省略下限或上限。
```
asset Vehicle extends Base {
  // An asset contains Fields, each of which can have an optional default value
  o String model default="F150"
  o String make default="FORD"
  o String reg default="ABC123"
  // A numeric field can have a range validation expression
  o Integer year default=2016 range=[1990,] optional // model year must be 1990 or higher
  o Integer[] integerArray
  o State state
  o Double value
  o String colour
  o String V5cID regex=/^[A-z][A-z][0-9]{7}/
  o String LeaseContractID
  o Boolean scrapped default=false
  o DateTime lastUpdate optional
  --> Participant owner //relationship to a Participant, with the field named 'owner'.
  --> Participant[] previousOwners optional // Nary relationship
  o Customer customer
}
```

## 导入

使用关键字`import`和一个完全限定类型名称可以从其他命名空间导入类型。或者使用`.*`符号从其他命名空间导入所有类型。
```
import org.example.MyAsset
import org.example2.*
```

## 装饰器

资源的资源和属性可以附加装饰器。装饰器被用来用元数据注解模型。下面的示例将`foo`装饰器添加到买方参与者，“arg1”和2作为参数传递给装饰器。

同样，装饰器可以附加到属性、关联和枚举值。
```
@foo("arg1", 2)
participant Buyer extends Person {
}
```

资源定义和属性可以用0个或多个装饰来装饰。请注意，每个元素类型只允许一个装饰器实例。也就是说，`@bar`装饰器在同一个元素上列出两次是无效的。

## 装饰器参数

装饰器可能有一个任意的参数列表（0个或多个项目）。参数值必须是字符串、数字或布尔值。

## 装饰器API

装饰器可以通过ModelManager的introspect API在运行时访问。这允许外部工具和实用程序使用Composer建模语言（Composer Modeling Language，CTO）文件格式来描述核心模型，同时使用足够的元数据为自己的目的进行装饰。

下面的例子获取附加到一个类声明的myField属性的foo装饰器的第三个参数：
```
const val = myField.getDecorator('foo').getArgu
```
