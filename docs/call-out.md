# 从交易处理函数中调用REST API

> 此功能的状态是实验性的。我们欢迎您对此功能的实用性的反馈。我们可能会在未来发展这个功能，使其更加通用。虽然我们将努力确保向后兼容性，但不能保证。

## 场景

在某些情况下，希望能够从交易处理函数中调用REST API。这使你可以将区块链中的复杂计算移出。调用REST API允许交易处理器功能将复杂或昂贵的计算外包给中央或peer托管的服务。

## 调用外部REST服务

`post(url,data)`函数可用于交易处理器函数，允许它们将概念、交易、资产或参与者传递给外部服务。数据被序列化为JSON，以`application/json`内容编码并使用HTTP POST将数据发送到URL 。

所有运行时环境都支持`post`函数：Web（playground）、Node.js（嵌入式）和Hyperledger Fabric v1.0。

## 处理结果

该`post`方法返回一个`Promise`JS对象，该对象包含远程服务器返回的结果。JS对象具有以下属性：
- statusCode：HTTP状态码  
- body：HTTP响应正文  

请注意，从200到300的HTTP响应码将作为已解决的Promise返回，而其他任何响应码都会导致Promise被拒绝。被拒绝的Promise可以使用`.catch`Promise处理程序处理。

如果可能，响应的`body`属性自动转换为JS对象，否则返回为`string`。

交易处理器函数可以选择调用`getSerializer().fromJSON(object)`函数将响应正文JS对象转换成Composer资产、参与者或交易。

## 例子
```javascript
/**
 * Handle a POST transaction, calling Node-RED running on Bluemix
 * @param {org.example.sample.PostTransaction} postTransaction - the transaction to be processed
 * @transaction
 * @return [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) a promise that resolves when transaction processing is complete
 */
function handlePost(postTransaction) {
    var url = 'https://composer-node-red.mybluemix.net/compute';

    return post( url, postTransaction)
      .then(function (result) {
        // alert(JSON.stringify(result));
          postTransaction.asset.value = 'Count is ' + result.body.sum;
          return getAssetRegistry('org.example.sample.SampleAsset')
          .then(function (assetRegistry) {
              return assetRegistry.update(postTransaction.asset);
          });
      });
}
```

## 幂等和共识

REST请求的结果应该是一个纯函数（冪等），以确保所有peer在相同的输入下获得相同的结果。如果这个限制被承认，那么共识失败的风险就会降到最低。

*每个*peer都会打调用REST服务。

## CORS

来自Web浏览器的HTTP请求需要符合CORS。这要求服务器设置为接收OPTIONS方法并返回适当的头响应。

## Docker网络解决方案

来自HLF链码容器的HTTP请求需要DNS解析和适当的网络。例如，对于默认的Docker容器，不能使用[http://localhost](http://localhost/)，因为在容器内这不会解析到宿主机。最简单的解决方法是使用可公开解析的DNS名称。
