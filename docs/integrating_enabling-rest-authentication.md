# 为REST服务器启用身份认证

REST服务器可以配置为对客户端进行身份认证。启用此选项时，客户端必须在允许调用REST API之前向REST服务器进行身份认证。

## 选择一个认证策略

REST服务器使用开源[Passport](http://passportjs.org/)身份认证中间件。REST服务器的管理员必须选择Passport策略来认证客户端。可以选择多个Passport策略，允许REST服务器的客户端选择首选的身份认证机制。Passport包括广泛的策略（在写作时为300+），包括社交媒体（Google、Facebook、Twitter）和企业（SAML，LDAP）策略的组合。

本文档的其余部分将演示如何使用`passport-github`策略来使用他们的GitHub ID对用户进行身份认证。`passport-github`通过执行以下命令来安装策略:
```bash
npm install -g passport-github
```

## 配置REST服务器以使用认证策略

在启用REST API身份认证之前，必须为REST服务器配置一个要使用的Passport策略列表。该配置包括要使用的策略的名称以及每个策略的单独配置。

为了配置`passport-github`策略，我们需要在GitHub上注册一个OAuth应用程序，并获取客户端ID和客户端密码。按照以下步骤在GitHub上注册OAuth应用程序:

1. 导航到[GitHub](https://github.com/)并使用你的用户名和密码登录。

2. 点击右上角的个人资料图片，然后从下拉菜单中点击**设置**。

3. 点击左侧栏的**开发人员设置**下的**OAuth应用程序**。

4. 点击**注册一个新的应用程序**。

5. 指定以下设置:

   - 应用程序名称:Composer

   - 主页:[http://localhost:3000/](http://localhost:3000/)
   
   - 应用程序描述:Composer的OAuth应用程序
   
   - 授权回调URL:[http://localhost:3000/auth/github/callback](http://localhost:3000/auth/github/callback)

6. 点击**注册申请**。

7. 记下**客户端ID**和**客户端密钥**的值。

使用环境变量`COMPOSER_PROVIDERS`来指定REST服务器的配置。要配置REST服务器，需要用步骤7中获取的值替换`REPLACE_WITH_CLIENT_ID`和`REPLACE_WITH_CLIENT_SECRET`，然后执行以下命令:
```bash
export COMPOSER_PROVIDERS='{
  "github": {
    "provider": "github",
    "module": "passport-github",
    "clientID": "REPLACE_WITH_CLIENT_ID",
    "clientSecret": "REPLACE_WITH_CLIENT_SECRET",
    "authPath": "/auth/github",
    "callbackURL": "/auth/github/callback",
    "successRedirect": "/",
    "failureRedirect": "/"
  }
}'
```

##启动 启用了REST API身份认证的REST服务器

一旦设置了环境变量`COMPOSER_PROVIDERS`，就可以使用该`-a true`参数启动启用了身份认证的REST服务器。启用身份认证后，客户端必须先进行身份认证，然后才能向业务网络发出任何请求。

例如，以下是作为开发者教程一部分部署的业务网络的命令，然而你可能需要为你的业务网络修改命令:
```bash
composer-rest-server -c admin@my-network -a true
```

现在，导航到[http://localhost:3000/explorer/](http://localhost:3000/explorer/)上的REST API浏览器。如果已成功启用身份认证，则任何尝试使用REST API浏览器调用其中一个业务网络REST API操作的尝试都会被拒绝，并显示一条`HTTP 401 Authorization Required`消息。

## 使用Web浏览器向REST服务器认证

此步骤取决于REST服务器正在使用的Passport策略的配置和行为。

1. （Web浏览器）通过导航到环境变量`COMPOSER_PROVIDERS`中定义的属性`authPath`的值，来向到REST服务器认证。在上面的例子中，这是[http://localhost:3000/auth/github](http://localhost:3000/auth/github)。

2. REST服务器会将身份认证重定向到GitHub以执行OAuth Web服务器身份认证流程。GitHub会询问身份认证是否要授权Composer应用程序访问身份认证的帐户。点击**授权**按钮。

3. 如果成功，GitHub会将身份认证重定向到REST服务器。

现在，导航到[http://localhost:3000/explorer/](http://localhost:3000/explorer/)上的REST API浏览器。尝试再次使用REST API浏览器调用其中一个业务网络REST API操作。这一次，调用应该能成功。

## 使用HTTP或REST客户端向REST服务器认证

当用户向REST服务器进行身份认证时，会生成一个唯一的访问令牌并将其分配给经过身份认证的用户。当用户使用web浏览器进行认证时，访问令牌被存储在用户网络浏览器的本地存储器中的cookie中。当经过身份认证的用户发出后续请求时，将从cookie中获取令牌，并且验证访问令牌而不是重新认证用户身份。

访问令牌可用于认证任何希望调用REST服务器的HTTP或REST客户端。当HTTP或REST客户端无法执行配置的Passport策略所需的身份认证流程时，这是需要的。例如，所有OAuth2 Web认证流程都需要使用Web浏览器导航到认证提供商网站。

为了使用访问令牌，必须首先使用Web浏览器来获取访问令牌。当你向REST服务器进行身份认证时，REST API浏览器会在[http:// localhost:3000 / explorer /上](http://localhost:3000/explorer/)的页面顶部显示访问令牌。默认情况下，访问令牌是隐藏的，但可以通过点击`Show`按钮来显示。访问令牌是一个长的字母数字字符串，例如:`e9M3CLDEEj8SDq0Bx1tkYAZucOTWbgdiWQGLnOxCe7K9GhTruqlet1h5jsw10YjJ`

一旦访问令牌已被获取，访问令牌可以被放入任何HTTP或REST请求来认证HTTP或REST客户端。传递访问令牌有两种选择 - 使用查询字符串参数或HTTP头。对于以下两个示例，请将字符串`xxxxx`替换为访问令牌的值。

查询字符串 - 将`access_token`查询字符串参数添加到所有HTTP或REST请求:
```bash
curl -v http://localhost:3000/api/system/ping?access_token=xxxxx
```

HTTP头 - 将`X-Access-Token`头添加到所有HTTP或REST请求:
```bash
curl -v -H 'X-Access-Token: xxxxx' http://localhost:3000/api/system/ping
```
