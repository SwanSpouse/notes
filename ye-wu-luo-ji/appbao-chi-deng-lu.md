# App保持登录

## 简介

在使用App时，一次登录后App如果不主动退出登录，App将会在很长一段时间内保持登录状态。让用户感觉到登录一次就不用每次输入登录密码才能进行登录了。

## 常见App保持登录的实现方式

利用Cookie机制实现

* 如果我们的App和后端通讯采用的http通讯方式，可以利用cookie技术进行登录状态保持。比如我们可以把sessionID和有效期保存在cookie中，发给前端App，前端App收到后保存在本地。当访问后端服务把sessionID和有效期作为参数传给后台进行认证。直到sessionID失效，用户都不需要重新登录。

前端保存用户和密码

* 利用用户名和密码保持登录是指用户在第一次登录成功时，把用户名和密码保存的本地，下次用户打开App时登录利用保存的用户名和密码在后台自动完成。这种方式需要考虑用户名和密码的安全问题，防止信息被破解。

Token方式

* token方式在app认证上用的比较普遍，App初始登录时，提交账号和密码数据给服务端，服务端根据定义的的策略生成一个token字符串，token字符串中可以包含用户信息、设备ID等信息以保证用户的唯一性。服务端并对token设置一定的期限。服务端把生成的token字符串传给客户端，客户端保存token字符串，并在接下来的请求中带上这个字符串。相对于在App本地token的安全性更高了。

## Token方式保持App登录具体实现

首先在登录成功、注册成功的时候，后台会根据用户的user\_id、device\_id 等信息生成用户的session\_id，并返回给APP。

APP在每次请求后台的时候会把session\_id带上。

后台会对APP请求中的session\_id进行校验。校验session\_id是否合法、是否过期。如果不合法，则提示让APP重新进行登录。

如果session\_id合法，则正常处理APP请求。

![](../.gitbook/assets/session.png)

## 安全问题说明

**Q:** 如果用户Alice的请求被劫持（假设劫持者是Eve）。Eve获取到了用户的session\_id那是不是说明Eve可以模拟Alice向后台发送请求了？  
**A:** 当然不行，因为session\_id的生成规则里面包含了Alice的device\_id，仅仅是模拟请求是无法通过后台校验的。

**Q:** 如果Eve可以通过模拟器模拟Alice的设备号呢？那它是否能够通过后台的校验呢？  
**A:** 同样不行，后台会对url进行签名。如果你不知道签名的加密规则，你是无法获取到session\_id这个参数的。

**Q:** 如果Eve知道签名的加密规则、知道设备号、知道session\_id呢？  
**A:** 还是不行。在对url参数进行签名加密之后，后台会对整个请求的数据进行一个整体的加密。所以如果你不知道解密方式的时候，是无法从请求中获取到任何信息的。

**Q:** 如果Eve知道了整体加密规则，解密方式，url签名方式，session\_id了呢。  
**A:** 那你基本可以对后台进行你想要进行的操作了。。。

