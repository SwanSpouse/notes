### Url参数的Sign签名

现在大多数APP和后端都是使用http、https来进行交互的。在这里对开放式API接口如何对数据安全进行保证进行简单的梳理和说明。

#### 常见的开放性API接口的安全性问题：

请求来源是否合安全\(对身份进行验证\)。

请求参数是否被篡改。

请求的唯一性\(不可复制性\)如何进行保证。

#### 不验证的方式

APP请求后台接口，向后台发送请求：[http://api.test.com/getproducts?param2=value2&param1=value1](http://api.test.com/getproducts?param2=value2&param1=value1)

这种方式简单粗暴且没有任何的验证信息。任何人都可以通过这个接口获取到所有的product信息，导致信息泄露。

#### 对Url进行签名认证

首先对上述请求加入签名验证。

* 给app分配secret，用来对请求进行加密。
* 在发送请求时，app使用secret对请求进行签名认证。

具体的签名流程：首先，app会对要发送请求的参数按照某种规则来进行排序，例如字典序。上面的参数排序后为param1=value1&param2=value2。然后使用secret对参数进行加密，形成签名。将签名当做参数拼在请求后面。如：[http://api.test.com/getproducts?param2=value2&param1=value1&sign=BCC7C71CF93F9CDBDB88671B701D8A3](http://api.test.com/getproducts?param2=value2&param1=value1&sign=BCC7C71CF93F9CDBDB88671B701D8A3)

后台在接收到请求之后，首先会对采用和app同样的方式，将所有参数进行排序，并加密。然后和客户端传过来的签名进行对比，只有通过校验的请求才能正常提供服务。否则则认为是非法请求。

如果仅仅是这样，还有一个问题就是，一旦你的请求被别人截获了。虽然我无法对参数进行篡改。但是我可以反复使用获取的请求来获取数据。这时就需要对请求的唯一性进行保证。

#### 请求的唯一性

在上述的请求中，加入时间戳，同时设定一段的过期时间，例如30s。这样就算是别人可以获取到完整的请求链接，有效期也仅仅是也有限的几十秒。

因为考虑到当发生错误的时候，我们可以使用用户的请求log复现当时的场景。所以这部分功能后台没有做。

#### 请求的隐私性

可能还会有人担心明文的传输不够安全，别人还是可以通过url请求看到请求参数的具体内容。如果有一些敏感信息或者关键信息，那么会造成一定的信息泄露。所以在对请求进行签名之后，还会对请求进行一个整体的加密。并在后面附上加密方式的别名。例如上述请求经过加密后变为[http://api.test.com/getproducts?data=xBASDJFLASDLFJXADFA1234124&en=a1，请求参数经过加密之后变为data参数的内容，en表示前端进行加密的方式。后台通过en可以得知前端使用的哪种加密方式来进行加密。在请求到达时，首先对data值进行解密，然后再验证签名。](http://api.test.com/getproducts?data=xBASDJFLASDLFJXADFA1234124&en=a1，请求参数经过加密之后变为data参数的内容，en表示前端进行加密的方式。后台通过en可以得知前端使用的哪种加密方式来进行加密。在请求到达时，首先对data值进行解密，然后再验证签名。)

#### reference:

* [https://blog.csdn.net/qq\_15901351/article/details/80175169](https://blog.csdn.net/qq_15901351/article/details/80175169)



