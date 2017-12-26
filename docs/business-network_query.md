# 查询和过滤业务网络数据

查询用于返回区块链世界状态的数据; 例如，你可以编写一个查询来返回指定年龄的所有司机，或具有指定名称的所有司机。`composer-rest-server`组件通过生成的REST API暴露命名查询。

查询是业务网络定义的可选组件，用一个查询文件（`queries.qry`）编写。

注意：使用Hyperledger Fabric v1.0时，Hyperledger Fabric必须配置为使用CouchDB持久化。

过滤器与查询类似，但使用LoopBack过滤器语法，只能使用Hyperledger Composer REST API发送。目前只支持`WHERE`LoopBack过滤器。其中支持的`WHERE`中的运算符是：**=**、**和**、**或**、**gt**、**gte**、**lt**、**lte**、**neq**。过滤去通过`GET`方法针对资产类型、参与者类型或交易类型发出调用; 然后提供参数给过滤器。过滤器从指定的类中返回结果集，而不会返回指定类的扩展类的结果集。

## 查询的类型

Hyperledger Composer支持两种类型的查询：命名查询和动态查询。命名查询在业务网络定义中指定，并由composer-rest-server组件暴露为GET方法。动态查询可以在事务处理器函数的运行时动态构建，也可以从客户端代码动态构建。

### 编写命名查询

查询必须包含说明和语句。查询描述是一个描述查询功能的字符串。查询语句包含控制查询行为的运算符和函数。

查询描述可以是任何描述性的字符串。查询语句必须包括`SELECT`运算符，可以选择`FROM`、`WHERE`、`AND`、`ORDER BY`、`SKIP`或`LIMIT`。

查询应采用以下格式：
```
query Q1{
  description: "Select all drivers older than 65."
  statement:
      SELECT org.acme.Driver
          WHERE (age>65)
}
```

#### 查询参数

查询可以使用`_$`语法嵌入参数。请注意，查询参数必须是原始类型（字符串、整数、双精度、长整型、布尔型、日期时间）、关联或枚举。

下面的命名查询是根据3个参数定义的：
```
query Q18 {
    description: "Select all drivers aged older than PARAM"
    statement:
        SELECT org.acme.Driver
            WHERE (_$ageParam < age)
                ORDER BY [lastName ASC, firstName DESC]
                    LIMIT _$limitParam
                        SKIP _$skipParam
}
```

查询参数通过composer-rest-server为命名查询创建的GET方法自动暴露。

有关Hyperledger Composer查询语言的详细信息，请参阅[查询语言参考文档](reference_query-language.md)。

### 使用API查询

查询可以通过调用*buildQuery*或*query * API 来调用。*BuildQuery* API需要在API输入中指定整个查询字符串。*query* API需要你指定要运行的查询的名称。

有关查询API的更多信息，请参阅[API文档](https://hyperledger.github.io/composer/api/api-doc-index.html)。

### 查询访问控制

在返回查询结果时，你的访问控制规则将应用于结果。当前用户无权查看的任何内容将从结果中删除。

例如，如果当前用户发送将返回所有资产的查询，如果他们只有权限查看部分资产，那么查询将仅返回部分资产。

## 使用过滤器

过滤器只能使用Hyperledger Composer REST API进行提交，并且必须使用[LoopBack语法](https://loopback.io/doc/en/lb2/Where-filter.html)。要提交查询，必须针对资产类型、参与者类型或交易类型提交**GET** REST调用，并将过滤器作为参数提供。过滤器参数支持的数据类型是*数字*、*布尔值*、*日期时间*和*字符串*。基本过滤器采用以下格式，其中`op`表示运算符：
```
{"where": {"field1": {"op":"value1"}}}
```

*请注意*：只有顶层`WHERE`运算符可以有两个以上的操作对象。

目前仅支持`WHERE`LoopBack过滤器。`WHERE`中支持的运算符是：**=**、**和**、**或**、**gt**、**gte**、**lt**、**lte**、**neq**。过滤器可以组合多个运算符，在以下示例中，一个**和**运算符嵌套在一个**or**运算符中。
```
{"where":{"or":[{"and":[{"field1":"foo"},{"field2":"bar"}]},{"field3":"foobar"}]}}
```

的**between**运算符返回给定范围的值。它接受数字、日期时间值和字符串。如果提供字符串，则**between**运算符将按字母顺序返回提供的字符串之间的结果。在下面的示例中，过滤器将返回司机属性在*a*和*c*之间(含)的所有资源（按字母顺序排列）。
```
{"where":{"driver":{"between": ["a","c"]}}}
```
