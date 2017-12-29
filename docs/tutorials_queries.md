## 使用Composer查询语言和REST API的查询教程

在本教程中，我们将在[开发者教程](tutorials_developer-tutorial.md)的基础上进行扩展，以展示查询。原生查询语言可以使用条件过滤返回的结果，并可以在交易中被调用以执行操作，例如更新或删除结果集上的资产。

查询在查询文件（`.qry`）中定义，文件位于业务网络定义的父目录中的。查询包含一个WHERE子句，它定义了选择资产或参与者的过滤条件。

本教程使用`tutorial-network`业务网络，该网络[开发者教程](tutorials_developer-tutorial.md)中被开发和部署。

### 先决条件

开始本教程之前：

- 完成[开发环境的安装](installing_development-tools.md)。

- 完成[开发者教程](tutorials_developer-tutorial.md)。

### 第一步：更新业务网络

开发者教程中创建的业务网络必须更新。更新的业务网络包含两个事件和一个额外的交易。

#### 更新模型文件

模型文件必须更新以包含事件和新交易。

1、打开业务网络`tutorial-network`的模型（`.cto`）文件。  
2、将以下事件和交易添加到模型中：

```
   event TradeNotification {
       --> Commodity commodity
   }

   transaction RemoveHighQuantityCommodities {
   }

   event RemoveNotification {
       --> Commodity commodity
   }
```

3、将更改保存到模型中。

#### 更新交易逻辑以使用查询和事件

现在，域模型已经更新，我们可以编写额外的业务逻辑，在提交交易处理时执行。在本教程中，我们将事件和查询添加到下面的业务逻辑中。

1、打开交易处理器函数文件`lib/logic.js`。  
2、用下面的JavaScript替换交易逻辑：  
```javascript
   /**
    * Track the trade of a commodity from one trader to another
    * @param {org.acme.biznet.Trade} trade - the trade to be processed
    * @transaction
    */
   function tradeCommodity(trade) {

       // set the new owner of the commodity
       trade.commodity.owner = trade.newOwner;
       return getAssetRegistry('org.acme.biznet.Commodity')
           .then(function (assetRegistry) {

               // emit a notification that a trade has occurred
               var tradeNotification = getFactory().newEvent('org.acme.biznet', 'TradeNotification');
               tradeNotification.commodity = trade.commodity;
               emit(tradeNotification);

               // persist the state of the commodity
               return assetRegistry.update(trade.commodity);
           });
   }

   /**
    * Remove all high volume commodities
    * @param {org.acme.biznet.RemoveHighQuantityCommodities} remove - the remove to be processed
    * @transaction
    */
   function removeHighQuantityCommodities(remove) {

       return getAssetRegistry('org.acme.biznet.Commodity')
           .then(function (assetRegistry) {
               return query('selectCommoditiesWithHighQuantity')
                       .then(function (results) {

                           var promises = [];

                           for (var n = 0; n < results.length; n++) {
                               var trade = results[n];

                               // emit a notification that a trade was removed
                               var removeNotification = getFactory().newEvent('org.acme.biznet', 'RemoveNotification');
                               removeNotification.commodity = trade;
                               emit(removeNotification);

                               // remove the commodity
                               promises.push(assetRegistry.remove(trade));
                           }

                           // we have to return all the promises
                           return Promise.all(promises);
                       });
           });
   }
```
3、保存你的更改到`logic.js`。

第一个函数`tradeCommodity`将在收到的Trade交易时更改商品（用一个新的拥有者参与者）的拥有者属性，并发出通知事件。然后，将修改后的商品(Commodity)保存回用于存储商品实例的资产库中。

第二个函数调用一个名为“selectCommoditiesWithHighQuantity”（在`queries.qry`中定义）的查询，它将返回数量大于60的所有商品资产记录; 发出一个事件; 并从AssetRegistry(资产库)中移除商品。

### 第二步：创建一个查询定义文件

交易处理器逻辑使用的查询被定义在一个叫做`queries.qry`的文件中。每个查询条目定义执行查询的资源和条件。

1、在`tutorial-network`目录中，创建一个名为`queries.qry`的新文件。  
2、将以下代码复制并粘贴到`queries.qry`：  
```javascript
   /** Sample queries for Commodity Trading business network
   */

   query selectCommodities {
     description: "Select all commodities"
     statement:
         SELECT org.acme.biznet.Commodity
   }

   query selectCommoditiesByExchange {
     description: "Select all commodities based on their main exchange"
     statement:
         SELECT org.acme.biznet.Commodity
             WHERE (mainExchange==_$exchange)
   }

   query selectCommoditiesByOwner {
     description: "Select all commodities based on their owner"
     statement:
         SELECT org.acme.biznet.Commodity
             WHERE (owner == _$owner)
   }

   query selectCommoditiesWithHighQuantity {
     description: "Select commodities based on quantity"
     statement:
         SELECT org.acme.biznet.Commodity
             WHERE (quantity > 60)
   }
```

3、保存你的更改到`queries.qry`。

### 第三步：重新生成你的业务网络档案

在更改业务网络中的文件后，业务网络必须重新打包为业务网络档案（`.bna`），并重新部署到Hyperledger Fabric实例。

1、使用命令行，导航到`tutorial-network`目录。  
2、运行以下命令：  
```bash
   composer archive create --sourceType dir --sourceName . -a tutorial-network@0.0.1.bna
```

### 第四步：部署更新的业务网络定义

我们需要部署修改后的网络，成为区块链上的最新版本！我们正在使用新创建的业务网络档案文件来更新现有的已部署业务网络; 这是我们在开发者教程中使用的同一个业务网络名称。

1、切换到终端，将目录切换到包含`tutorial-network.bna`的文件夹。  
2、运行以下命令更新业务网络：  
```bash
   composer network update -a tutorial-network@0.0.1.bna -c admin@tutorial-network
```

3、运行以下命令测试网络是否已部署：  
```bash
   composer network ping -c admin@tutorial-network
```

### 第五步：为更新后的业务网络重新生成REST API

现在，我们把刚更新的业务网络与添加的查询集成在一起，并为这个业务网络暴露REST API。

1、使用命令行，导航到`tutorial-network`目录。  
2、使用以下命令启动REST服务器：  
```bash
   composer-rest-server
```
3、输入`admin@tutorial-network`作为卡片名称。  
4、当询问是否在生成的API中使用名称空间时，请选择**不使用名称空间**。  
5、当询问是否保护生成的API时选择**否**。  
6、当询问是否启用事件发布时，选择**是**。  
7、当询问是否启用TLS安全时，请选择**否**。  

### 第六步：测试REST API并创建一些数据

打开Web浏览器并导航到[http：//localhost:3000/explorer](http://localhost:3000/explorer)。你应该看到LoopBack API浏览器，允许你查看和测试生成的REST API。

我们应该能够看到被添加了名为“Query”的REST端点，并且在展开时显示了业务网络 `tutorial-network`中定义的REST查询操作列表

![作为REST端点公开的查询](https://hyperledger.github.io/composer/assets/img/tutorials/query/rest-explorer-discover.png)

在开始之前，我们需要创建一些数据，以充分展示查询。使用提供的示例JSON数据，使用REST API创建3个交易者（参与者）和更多商品（资产）。

1、首先，在REST Explorer中点击'Trader'，然后点击/Trader上的'POST'方法，然后向下滚动到Parameter部分 - 依次创建下面的Trader实例：
```json
   {
     "$class": "org.acme.biznet.Trader",
     "tradeId": "TRADER1",
     "firstName": "Jenny",
     "lastName": "Jones"
   }
```

2、点击“Try it out”来创建参与者。“Response Code”（向下滚动）应该是200（成功）  
3、通过复制以下JSON创建另一个交易者：  
```json
   {
     "$class": "org.acme.biznet.Trader",
     "tradeId": "TRADER2",
     "firstName": "Jack",
     "lastName": "Sock"
   }
```

4、通过复制以下JSON创建第三个交易者：  
```json
   {
     "$class": "org.acme.biznet.Trader",
     "tradeId": "TRADER3",
     "firstName": "Rainer",
     "lastName": "Valens"
   }
```

5、现在滚动到顶部，并在REST浏览器中点击“Commodity”对象。  
6、点击POST操作，并向下滚动到Parameters 部分：以上述相同的方式，为拥有者TRADER1和TRADER2创建两个商品资产记录（见下文）：  
```json
{
  "$class": "org.acme.biznet.Commodity",
  "tradingSymbol": "EMA",
  "description": "Corn",
  "mainExchange": "EURONEXT",
  "quantity": 10,
  "owner": "resource:org.acme.biznet.Trader#TRADER1"
}
```
```json
{
  "$class": "org.acme.biznet.Commodity",
  "tradingSymbol": "CC",
  "description": "Cocoa",
  "mainExchange": "ICE",
  "quantity": 80,
  "owner": "resource:org.acme.biznet.Trader#TRADER2"
}
```

### 第七步：使用商品交易REST API浏览器执行查询

现在我们有了一些资产和参与者，我们可以使用生成的Query REST操作来测试一些查询。

#### 执行一个简单REST查询

现在我们有资产和参与者，我们可以尝试一些查询。

我们可以首先尝试的最简单的REST查询是我们的命令查询`selectCommodities`。

展开“Query”REST端点，你将看到我们在模型中定义的命名查询。

这些查询现在暴露为REST查询，并为每个生成一个/GET操作。请注意，查询的描述（我们在模型定义中定义的）显示在右侧。

![商品：REST端点](https://hyperledger.github.io/composer/assets/img/tutorials/query/commodity-rest-endpointhdr.png)

1、展开`selectCommodities`查询。  
2、点击“Try it Out”按钮。  

![设置REST查询：选择所有商品](https://hyperledger.github.io/composer/assets/img/tutorials/query/query-select-commodities.png)

它将返回所有现有的商品 - 应该有2个资产返回。

![查询结果：所有商品](https://hyperledger.github.io/composer/assets/img/tutorials/query/query-commodity-results.png)

### 执行过滤的REST查询

让我们通过他们的交易所选择商品 - 例如“EURONEXT”的主交易所。

1、展开查询端点“selectCommoditiesByExchange”并滚动到“Parameters”部分。  
2、在“Exchange”参数中输入“EURONEXT”。  
3、点击“Try it Out”。  

![由Exchange安装REST查询](https://hyperledger.github.io/composer/assets/img/tutorials/query/query-selectby-exchange.png)

结果显示，只有那些与“EURONEXT”交易所的商品才会显示在响应正文中

![查询结果：商品交换](https://hyperledger.github.io/composer/assets/img/tutorials/query/queryresults-selectby-exchange.png)

### 使用命名查询的结果执行交易修改

最后，你会记得我们已经定义了一个简单的查询，用于在我们的查询文件中筛选数量大于60的商品。在交易函数中使用时查询非常强大，例如允许交易逻辑查询出一组资产或参与者来执行更新或删除操作。

![重新搜索查询定义](https://hyperledger.github.io/composer/assets/img/tutorials/query/querydef-recall-high-qty-commodities.png)

我们在交易`removeHighQuantityCommodities`中使用`selectCommoditiesWithHighQuantity`查询。如果你在REST浏览器中执行这个/GET操作，你将看到它仅选择数量大于60的资产。

![使用查询重新记录交易逻辑](https://hyperledger.github.io/composer/assets/img/tutorials/query/functiondef-recall-high-qty-commodities.png)

现在，让我们使用查询来执行一个针对大数量商品的删除。

首先检查自己有多少商品（使用'Commodity'/GET操作），你应该看到至少两个商品，其中一个商品（Cocoa）的数量大于60。

![商品REST端点](https://hyperledger.github.io/composer/assets/img/tutorials/query/commodity-rest-endpointhdr.png)![“删除交易”调用之前的结果](https://hyperledger.github.io/composer/assets/img/tutorials/query/txn-query-commodity-pre-rm.png)

让我们看看实际的查询，通过点击REST Endpoint `/selectCommoditiesWithHighQuantity`然后点击/GET，然后向下滚动到“Try it Out” - 应该有一个符合条件的商品。

![查询大量商品](https://hyperledger.github.io/composer/assets/img/tutorials/query/query-selectby-comm-high-qty.png)

好。现在让我们执行一个REST交易，它使用我们的“大数量”查询定义来决定删除哪些商品。

单击RemoveHighQuantityCommodities REST端点以显示相同的/POST操作。

![显示RemoveHighQuantityCommodities端点](https://hyperledger.github.io/composer/assets/img/tutorials/query/txn-show-rm-comm-high-qty.png)

点击POST，向下滚动到参数部分，并点击“Try it Out” -请注意：你*不是*必须在“数据”部分输入数据。

向下滚动，你应该看到一个transactionId，它代表交易处理器函数中的“remove”调用（本身就是一个区块链交易），它将更新世界状态 - 响应码应该是200

![进行“高数量”清除交易](https://hyperledger.github.io/composer/assets/img/tutorials/query/txn-exec-rm-comm-high-qty.png)

最后，让我们来验证我们的商品状态。返回到“Commodity”REST操作并再次执行/GET操作....“Try it Out”。

结果应该显示，商品资产“Cocoa”已经消失，即只有数量<= 60的商品资产仍然存在，即我们的例子中的资产“Corn”。命名查询为交易更新提供了输入（以删除大数量商品），并在业务逻辑中执行。

![查询驱动的交易函数的最终结果](https://hyperledger.github.io/composer/assets/img/tutorials/query/txn-query-commodities-post-rm.png)

### 恭喜！

干得好，现在你已经完成了这个教程，我们希望你现在对Composer中查询的能力有了更好的理解。你可以开始创建/构建你自己的查询（或修改现有查询并将相关数据添加到此业务网络 - 请注意：任何查询更改都需要重新部署）才能试用！
