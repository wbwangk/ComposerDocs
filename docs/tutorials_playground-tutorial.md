## Playground 教程

在这个手把手教程中，我们将通过建立一个商业网络，定义我们的资产、参与者和交易，以及通过创建一些参与者和资产来测试我们的网络，并提交交易来将资产的所有权从一个更改为另一个。

### 第一步：打开Hyperledger Composer Playground

打开[Composer Playground](https://composer-playground.mybluemix.net/)。你应该看到“ **我的商业网络”**屏幕。“ **我的商业网络”**页面显示了你可以连接到的商业网络的摘要，以及可用于连接的身份。现在不要担心这个问题，因为我们要建立自己的网络。

### 第二步：创建一个新的商业网络

接下来，我们要从头开始创建一个新的商业网络。一个商业网络有几个定义的属性; 一个名字和一个可选的描述。你也可以选择在现有模板上建立新的商业网络，或导入自己的模板。

1. 点击在Web浏览器标题下**部署一个新的商业网络**开始。
2. 新的商业网络需要一个名字，让我们叫它`tutorial-network`。
3. 你也可以选择输入商业网络的说明。
4. 接下来我们必须选择一个我们基于的商业网络，因为我们要从头构建网络，点击**空白商业网络**。
5. 现在我们的网络已经定义好了，点击**部署**。

### 第三步：连接到商业网络

现在我们已经创建并部署了商业网络，你应该会看到一个名为*admin*的新商业网络卡片，用于你的钱包中的商业网络* tutorial-network*。钱包可以包含商业网络卡片以连接到多个部署的商业网络。

连接到外部区块链时，商业网络卡片代表连接到商业网络所需的一切。它们包括连接详细信息、身份认证材料和元数据。

要连接到我们的商业网络，请点击我们商业网络卡下面的** 现在连接**。

### 第四步：添加一个模型文件

正如你所看到的，我们现在处于“ **定义”**选项卡中，在部署和测试（使用**测试**选项卡）商业网络之前，此选项卡是创建和编辑商业网络定义文件的地方。

当我们选择一个空的商业网络模板时，我们需要定义我们的商业网络文件。第一步是添加一个模型文件。模型文件定义了我们商业网络中的资产、参与者、交易和事件。

有关我们建模语言的更多信息，请查看我们的[文档](https://hyperledger.github.io/composer/stable/reference/cto_language.html)。

1. 点击**添加一个文件**按钮。

2. 点击**模型文件**，然后点击**添加**。

3. 删除模型文件中的代码行并用以下替换：
   ```
   /**
    * My commodity trading network
    */
   namespace org.acme.mynetwork
   asset Commodity identified by tradingSymbol {
       o String tradingSymbol
       o String description
       o String mainExchange
       o Double quantity
       --> Trader owner
   }
   participant Trader identified by tradeId {
       o String tradeId
       o String firstName
       o String lastName
   }
   transaction Trade {
       --> Commodity commodity
       --> Trader newOwner
   }
   ```

   这个领域模型定义了一个资产类型`Commodity`和单个参与者类型`Trader`以及一个用于修改商品所有者的交易类型`Trade`。

### 第五步：添加一个交易处理器脚本文件

现在已经定义了领域模型，我们可以定义商业网络的交易逻辑。Composer使用JavaScript函数为一个商业网络表达逻辑。当一个交易提交处理时这些函数会自动执行。

有关编写交易处理函数的更多信息，请查阅我们的[文档](https://hyperledger.github.io/composer/stable/reference/js_scripts.html)。

1. 点击**添加文件**按钮。

2. 单击**脚本文件**，然后单击**添加**。

3. 删除脚本文件中的代码行并替换为以下代码：
   ```javascript
   /**
    * Track the trade of a commodity from one trader to another
    * @param {org.acme.mynetwork.Trade} trade - the trade to be processed
    * @transaction
    */
   function tradeCommodity(trade) {
       trade.commodity.owner = trade.newOwner;
       return getAssetRegistry('org.acme.mynetwork.Commodity')
           .then(function (assetRegistry) {
               return assetRegistry.update(trade.commodity);
           });
   }
   ```

这个函数只是简单地改变一个商品的`owner`属性，根据一个传入的`Trade`交易的`newOwner`属性。然后它将修改后`Commodity`持久化到资产库中，用于存储`Commodity`实例。

### 第六步：访问控制

访问控制文件定义了商业网络的访问控制规则。我们的网络很简单，所以默认的访问控制文件不需要编辑。基本文件给予了当前参与者`networkAdmin`商业网络和系统级操作的完全访问权限。

虽然你可以有多个模型或脚本文件，但在任何商业网络中只能有一个访问控制文件。

有关访问控制文件的更多信息，请查看我们的[文档](https://hyperledger.github.io/composer/stable/reference/acl_language.html)。

### 第七步：部署更新后的商业网络

现在我们有模型、脚本和访问控制文件，我们需要部署和测试我们的商业网络。

点击**更新**将更新部署到我们的商业网络。  
[视频]（https://hyperledger.github.io/composer/stable/assets/img/tutorials/playground/deploy_updates_render.mp4）  
### 第八步：测试商业网络的定义

接下来，我们需要测试我们的商业网络，创建一些参与者（本例中是*交易者*）、创建了一个资产（一个*商品*），然后使用我们的*Trade*交易改变*商品*的所有权。

点击**测试**标签开始。  
[视频]（https://hyperledger.github.io/composer/stable/assets/img/tutorials/playground/test_tab_render.mp4)  

### 第九步：创建参与者

我们应该添加到商业网络的第一件事是两个参与者。

1. 确保左边选择了**交易者**选项卡，然后点击右上角的“ **创建新参与者** ”。

2. 你可以看到*交易者*参与者的数据结构。我们需要一些容易识别的数据，所以删除那里的代码并粘贴下面的代码：
   ```json
   {
     "$class": "org.acme.mynetwork.Trader",
     "tradeId": "TRADER1",
     "firstName": "Jenny",
     "lastName": "Jones"
   }
   ```

3. 点击**新建**创建参与者。

4. 你应该能够看到你创建的新*交易者*参与者。我们需要另一个*交易者*来测试我们的*Trade *交易，所以创建另一个*交易者*，但这次使用以下数据：
   ```json
   {
     "$class": "org.acme.mynetwork.Trader",
     "tradeId": "TRADER2",
     "firstName": "Amy",
     "lastName": "Williams"
   }
   ```

在继续之前，确保两个参与者都存在于*交易者*视图中！  
[视频]（https://hyperledger.github.io/composer/stable/assets/img/tutorials/playground/create_new_participant_render.mp4)  

### 第十步：创建一个资产

现在我们有两个*交易者*参与者，我们需要一些交易。创建资产与创建参与者非常相似。我们正在创建的*商品*我有一个*所有者*属性，表明它属于*tradeId*为`TRADER1`的*交易者*。

1. 点击**资产**下的**商品**标签，然后点击**创建新资产**。********

2. 删除资产数据并将其替换为以下内容：
   ```json
   {
     "$class": "org.acme.mynetwork.Commodity",
     "tradingSymbol": "ABC",
     "description": "Test commodity",
     "mainExchange": "Euronext",
     "quantity": 72.297,
     "owner": "resource:org.acme.mynetwork.Trader#TRADER1"
   }
   ```

3. 创建此资产后，你应该能够在**商品**选项卡中看到它。  
[视频](https://hyperledger.github.io/composer/stable/assets/img/tutorials/playground/create_new_asset_render.mp4)  

### 第十一步：在参与者之间转移商品

现在，我们有两个*交易者*和一个用于交易的*商品*，我们可以测试我们的*Trade*交易。

交易是Hyperledger Composer商业网络中所有变化的基础，如果你想在本教程后践行自己的尝试，请尝试从**我的商业网络**屏幕创建另一个商业网络，并使用更高级的商业网络模板。

测试*Trade*交易：

1. 点击左侧的**提交交易**按钮。

2. 确保交易类型是*Trade*。

3. 将交易数据替换为以下内容，或者仅更改细节：
   ```json
   {
     "$class": "org.acme.mynetwork.Trade",
     "commodity": "resource:org.acme.mynetwork.Commodity#ABC",
     "newOwner": "resource:org.acme.mynetwork.Trader#TRADER2"
   }
   ```

4. 点击**提交**。

5. 通过展开资产数据部分，检查我们的资产已经改变所有权从`TRADER1`到`TRADER2`。你应该看到，所有者被列为`resource:org.acme.mynetwork.Trader#TRADER2`。

6. 要查看我们商业网络的完整交易记录，请点击左侧的**所有交易**。这里是每个被提交的交易的清单。你可以看到，我们使用用户界面执行的某些操作，如创建*Trader*参与者和*商品*资产，被记录为交易，即使它们没有在我们的商业网络模型中定义为交易。这些交易被称为“系统交易”，对所有商业网络都是通用的，并在Hyperledger Composer运行时中定义。  
[视频]（https://hyperledger.github.io/composer/stable/assets/img/tutorials/playground/submit_transaction_render.mp4）  

### 注销商业网络

现在交易已经成功运行，我们应该退出商业网络，在一开始的**我的商业网络**屏幕上退出。

1. 在屏幕的右上方是一个标有**admin**的按钮。这将列出你的当前身份，去登出，单击**admin**打开下拉菜单，然后单击**我的商业网络**。

### 接下来是什么？

你已完成最初的Hyperledger Composer Playground教程，你可能需要使用其他示例或模板开始构建自己的商业网络。

你可能想要尝试[**开发者教程**](https://hyperledger.github.io/composer/stable/tutorials/developer-tutorial.html)，获取你的本地开发环境设置，生成REST API和框架Web应用程序。

或者，你可以[运行本地](https://hyperledger.github.io/composer/stable/installing/using-playground-locally.html)连接到Hyperledger Fabric的实例的Hyperledger [Composer Playground](https://hyperledger.github.io/composer/stable/installing/using-playground-locally.html)。
