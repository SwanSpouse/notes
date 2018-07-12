### OAuth2 单点登录

OAuth是一个关于授权（authorization）的开放网络标准，在全世界得到广泛应用，目前的版本是2.0版。

OAuth的作用就是让"客户端"安全可控地获取"用户"的授权，与"服务商提供商"进行互动。

OAuth在"客户端"与"服务提供商"之间，设置了一个授权层（authorization layer）。"客户端"不能直接登录"服务提供商"，只能登录授权层，以此将用户与客户端区分开来。"客户端"登录授权层所用的令牌（token），与用户的密码不同。用户可以在登录的时候，指定授权层令牌的权限范围和有效期。

"客户端"登录授权层以后，"服务提供商"根据令牌的权限范围和有效期，向"客户端"开放用户储存的资料。

#### OAuth2运行流程

![](/assets/OAuth2.png)

\(A）用户打开客户端以后，客户端要求用户给予授权。

（B）用户同意给予客户端授权。

（C）客户端使用上一步获得的授权，向认证服务器申请令牌。

（D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。

（E）客户端使用令牌，向资源服务器申请获取资源。

（F）资源服务器确认令牌无误，同意向客户端开放资源。

#### OAuth2的客户端的授权模式

1、授权码模式（authorization code）\(获取code、code换取access\_token\)

2、简化模式（implicit）\(直接换取access\_token，基本不用\)

3、密码模式（resource owner password credentials）\(客户端像用户索取账号密码，然后客户端向服务端索取授权，基本不用\)

4、客户端模式（client credentials）（客户端以自己的名义要求"服务提供商"提供服务；场景：提供接口服务\)

授权码模式（authorization code）是功能最完整、流程最严密的授权模式。它的特点就是通过客户端的后台服务器，与"服务提供商"的认证服务器进行互动。

![](/assets/authroization_code.png)

（A）用户访问客户端，后者将前者导向认证服务器。

（B）用户选择是否给予客户端授权。

（C）假设用户给予授权，认证服务器将用户导向客户端事先指定的"重定向URI"（redirection URI），同时附上一个授权码。

（D）客户端收到授权码，附上早先的"重定向URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。

（E）认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）

#### 各步骤说明

A步骤中，客户端申请认证的URI，包含以下参数：

* response\_type：表示授权类型，必选项，此处的值固定为"code"
* client\_id：表示客户端的ID，必选项
* redirect\_uri：表示重定向URI，可选项
* scope：表示申请的权限范围，可选项
* state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。

```shell
GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz
    &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
Host: server.example.com
```

C步骤中，服务器回应客户端的URI，包含以下参数：

* code：表示授权码，必选项。该码的有效期应该很短，通常设为10分钟，客户端只能使用该码一次，否则会被授权服务器拒绝。该码与客户端ID和重定向URI，是一一对应关系。
* state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。

D步骤中，客户端向认证服务器申请令牌的HTTP请求，包含以下参数：

* grant\_type：表示使用的授权模式，必选项，此处的值固定为"authorization\_code"。
* code：表示上一步获得的授权码，必选项。
* redirect\_uri：表示重定向URI，必选项，且必须与A步骤中的该参数值保持一致
* client\_id：表示客户端ID，必选项。

E步骤中，认证服务器发送的HTTP回复，包含以下参数：

* access\_token：表示访问令牌，必选项。
* token\_type：表示令牌类型，该值大小写不敏感，必选项，可以是bearer类型或mac类型。
* expires\_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。
* refresh\_token：表示更新令牌，用来获取下一次的访问令牌，可选项。
* scope：表示权限范围，如果与客户端申请的范围一致，此项可省略

#### 使用OAuth2样例

使用OAuth2来进行授权登录的例子。

![](/assets/OAuth2Process.png)

#### 关于OAuth2流程中两次申请的问题

在OAuth2标准中，需要两次向授权服务器进行申请。第一次申请code，第二次申请token。为什么不直接申请一个token。

* 豆瓣告诉用户几个信息，QQ授权中心的地址，授权后的回调地址，豆瓣的ClientID（QQ认证中心需要知道认证调用方的ID），用户拿着这几个东西去QQ验证密码后，得到TOKEN，然后交给豆瓣。豆瓣拿着TOKEN、ClientID、ClientSecret去QQ认证中心获取头像和姓名。

* 上面说发的矛盾在于认证服务器和资源服务器是同一家。都是QQ。但是当资源服务器和认证服务器不是一家的时候。就行不通了。豆瓣不能告诉资源服务器自己的ClientSecret。只能告诉资源服务器ClientId和Token，然后资源服务器会找认证服务器去验证Token是否有效。

#### reference

* [http://www.ruanyifeng.com/blog/2014/05/oauth\_2\_0.html](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
* [https://www.cnblogs.com/flashsun/p/7424071.html](https://www.cnblogs.com/flashsun/p/7424071.html) 



