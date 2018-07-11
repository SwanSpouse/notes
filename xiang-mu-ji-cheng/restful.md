### RESTful

REST全称是Representational State Transfer，中文意思是表述性状态转移。REST指的是一组架构约束条件和原则。" 如果一个架构符合REST的约束条件和原则，我们就称它为RESTful架构。

REST本身并没有创造新的技术、组件或服务，而隐藏在RESTful背后的理念就是使用Web的现有特征和能力， 更好地使用现有Web标准中的一些准则和约束。虽然REST本身受Web技术的影响很深， 但是理论上REST架构风格并不是绑定在HTTP上，只不过目前HTTP是唯一与REST相关的实例。 所以我们这里描述的REST也是通过HTTP实现的REST。

**资源（Resources）**URL定位资源,每一个URI代表一种资源;

**用HTTP动词（GET,HEAD,POST,PUT,PATCH,DELETE）描述操作**,客户端通过四个HTTP动词，对服务器端资源进行操作，实现”表现层状态转化；

用**状态码**表示操作结果。

#### RESTful的优势

RESTful API有助于客户端和服务端的功能分离，服务器完全扮演着一个“资源服务商”的角色。各种不同的客户端都可以通过同一套API与这个“资源服务商”交流，从而与资源进行互动；

设计API工作量减少，资源的操作种类有限（GET、POST…），API的一致性、自我描述性很强，不需要过多解释；

一致的URL和HTTP动词使用：确保系统能够接纳多样而又标准的客户端，保证客户端的演化能力。

无状态：保证了系统的横向拓展能力、服务端的演化能力。

HATEOAS：保证了应用本身的演化能力\(功能增加、改变\)。

#### URL资源定位

URL用来指定一个资源，资源就是服务器上的数据，不应该包含动词，由名词组成，且推荐用复数。

资源地址使用嵌套的结构，/api/users/csr/blogs表示’csr’的所有博客，/api/users/csr/blogs/1234567表示其中的某一篇博客。这些都是资源，后者嵌套在前者之中。

API versioning:可以放在URL里面，也可以用HTTP的header版本号可以放置到url中:https://example.org/api/v1/。

#### 用HTTP动词描述操作

对这个资源\(URL\)使用不同的HTTP方法，就代表对这个资源的不同操作：

* GET 获取资源

* HEAD 获取资源的概况\(响应的HTTP只有head，没有body\)

* POST 新建资源

* PUT 更新资源\(客户端提供完整资源数据\)

* PATCH 更新资源\(客户端提供需要修改的资源数据\)

* DELETE 删除资源

GET、HEAD、PUT、DELETE方法是幂等方法\(对于同一个内容的请求，发出n次的效果与发出1次的效果相同\)。

GET、HEAD方法是安全方法\(不会造成服务器上资源的改变\), HEAD 和 GET请求代表获取数据，必须是安全、幂等的，

PATCH不一定是幂等的。PATCH的实现方式有可能是”提供一个用来替换的数据”，也有可能是”提供一个更新数据的方法”\(比如data++\)。如果是后者，那么PATCH不是幂等的。

PUT是幂等方法，POST不是。所以PUT用于更新、POST用于新增比较合适。

#### 状态转化, 用状态码表示操作结果

互联网通信协议HTTP协议，是一个无状态协议，所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生”状态转化”（State Transfer）。而这种转化是建立在表现层之上的，所以就是”表现层状态转化”。

客户端用到的手段，只能是HTTP协议。通过GET、POST、PUT、DELETE操作资源：获取、新建、更新、删除

200 OK - \[GET\]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。 

201 CREATED -\[POST/PUT/PATCH\]：用户新建或修改数据成功。 

202 Accepted -\[\*\]：表示一个请求已经进入后台排队（异步任务） 

204 NO CONTENT - \[DELETE\]：用户删除数据成功。 

400 INVALID REQUEST -\[POST/PUT/PATCH\]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的

401 Unauthorized -\[\*\]：表示用户没有权限（令牌、用户名、密码错误）。 

403 Forbidden -\[\*\]表示用户得到授权（与401错误相对），但是访问是被禁止的。 

404 NOT FOUND -\[\*\]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。

406 Not Acceptable -\[GET\]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。

410 Gone -\[GET\]：用户请求的资源被永久删除，且不会再得到的。

422 Unprocesable entity -\[POST/PUT/PATCH\]当创建一个对象时，发生一个验证错误。 

500 INTERNAL SERVER ERROR -\[\*\]：服务器发生错误，用户将无法判断发出的请求是否成功。

#### reference

* http://www.runoob.com/w3cnote/restful-architecture.html



