### web框架

#### web框架的功能

先明确一下web框架应该提供的功能，然后对照着。看看在backend中是怎么来进行实现的。

首先作为一个web框架，应该解决的问题有：

* 路由：如何将请求的 URL 映射到处理它的代码上？
* 模板：怎样动态地构造请求的 HTML 返回给客户端。
* cookie和session
* ORM
* 并发控制等

#### backend 中的路由

没有显式的注册路由的地方。通过反射机制来实现。

首先在service定义方法GetUserInfo，其对应的URL为/user/info。在启动时，后台会通过反射的方法获取service包下的所有方法，如果是以Get开头的，则对应是Get方法；同样的如果以Post开头的，对应的就是Post方法；以HTML开头的，则返回的是HTML页面。这样就不用手动去将URL和处理它的函数相关联起来。实现一个方法，则自动关联了相应的URL。

#### backend中的模板

这个没有什么特别的实现，采用的就是golang原生的。

#### cookie和session

cookie在后台项目中很少用到，一般都是在H5页面用到。

后台会验证客户端的session，看是否合法或者过期。后台的sessionID是根据UserId、时间戳、随机数和来源（H5、APP）生成的。

```go
func GenerateSessionId(uid int64, deviceModel, source string) string {
	session := Session{
		UserId:    uid,
		Timestamp: time.Now().Unix(),
		RandNum:   GenerateSessionRandNumber(uid, deviceModel, source),
		Source:    source,
	}
	output, err := json.Marshal(session)
	if err != nil {
		panic(err)
	}
	en, err := util.TripleDesEncrypt(output, a.key)
	if err != nil {
		panic(err)
	}
	return hex.EncodeToString(en)
}
```

所以在用户访问需要验证sessionID的url时，会将session解析出来，然后进行验证是否和合法、过期等。

#### backend中的ORM

目前后台使用的ORM框架是自己实现的，代码也相对来说比较简单，实现了一些基本的功能。包括基本的对象到数据库表的映射、关联表到对象的查询、各种数据库的查询接口。

映射关系

* 数据库的表名和列名以每个首字母为大写的单词进行分割，以'\_'相连，转换成小写字母后，对应于数据库中的表名和列名。

tag说明

* 对于前面所述的对象，每个域之后都会跟上一些标签，如：`pk:"true" ai:"true"`，这些标签对应于当前ORM的一些操作。

当前ORM中支持的一些操作如下表所示：

| 标签 | 含义 | 取值 |
| :--- | :--- | :--- |
| pk | 标识是否为主键 | true和false，如pk:"true" |
| ai | 标识是否为自增列 | true和false，如ai:"true" |
| ignore | 是否忽略该列的值 | true和false，如ignore:"true" |
| or | 将该属性配置的对应对象取出，一般配合table使用 | has\_many、has\_one、belongs\_to，如or:"has\_one" table:"test" |

  
注：

* 对于pk、ai的列，进行数据库插入操作后，ORM会将主键的id放入对象中，因此代码可以直接引用。
* 对于ignore的列，一般用于数据库的时间戳的更新。
* 对于or的使用，一般用于拿出关联的表的数据，放于对象中，其中两个表是根据主键进行关联的。















