# Hyperledger Composer查询语言

Hyperledger Composer中的查询以定制的查询语言编写。查询在业务网络定义中的一个叫（`queries.qry`）的查询文件中定义。

## 查询语法

所有查询都必须包含`description`和`statement`属性。

### 描述

该`description`属性是描述查询功能的字符串。它必须包含但可以包含任何东西。

### 声明

该`statement`属性包含查询的定义规则，可以具有以下运算符：

- `SELECT` 是一个强制性操作符，默认情况下定义要返回的库和资产或参与者类型。
- `FROM` 是一个可选的运算符，它定义了一个不同的库来查询
- `WHERE` 是可选运算符，它定义了要应用于库数据的条件。
- `AND` 是定义附加条件的可选运算符。
- `OR` 是一个可选的操作员，它定义了选择性条件
- `CONTAINS` 是一个可选的运算符，它为数组值定义条件
- `ORDER BY` 是一个可选的运算符，它定义了结果的排列顺序。
- `SKIP` 是一个可选的运算符，它定义了要跳过的结果数量。
- `LIMIT` 是一个可选的运算符，它定义了从查询返回结果的最大数量，默认情况下，limit设置为25。

#### 示例查询

这个查询从默认库中返回所有符合条件的司机，要求年龄小于输入参数，*或者* firstName为“Dan” 的，并且lastName不能是"Selman"。

实际上，这个查询返回lastName不是“Selman”的所有司机，只要它们在定义的年龄之下，或者fistNameu是Dan，并且按lastName升序和firstName降序来排序结果。
```
query Q20{
    description: "Select all drivers younger than the supplied age parameter or who are named Dan and whose lastName is not Selman, ordered by lastname, firstname"
    statement:
        SELECT org.acme.Driver
            WHERE ((age < _$ageParam OR firstName == 'Dan') AND (lastName != 'Selman'))
                ORDER BY [lastName ASC, firstName DESC]
}
```

#### 查询中的参数

查询可以用未定义参数来编写，执行时参数必须填充值。例如，以下查询将返回*年龄*属性大于输入参数的所有司机：
```
query Q17 {
    description: "Select all drivers aged older than PARAM"
    statement:
        SELECT org.acme.Driver
            WHERE (_$ageParam < age)
}
```

## 接下来是什么？

- [将查询应用到业务网络。](https://hyperledger.github.io/composer/business-network/query.html)
- [从交易中发出事件。](https://hyperledger.github.io/composer/business-network/publishing-events.html)
- [Hyperledger Composer API文档。](https://hyperledger.github.io/composer/api/api-doc-index.html)
