### NSQ



#### features

* **Distributed : **NSQ提供了分布式的，去中心化，且没有单点故障的拓扑结构，稳定的消息传输发布保障，能够具有高容错和HA（高可用）特性。
* **Scalable易于扩展:**NSQ支持水平扩展，没有中心化的brokers。内置的发现服务简化了在集群中增加节点。同时支持pub-sub和load-balanced 的消息分发。
* **Ops Friendly **NSQ非常容易配置和部署，生来就绑定了一个管理界面。二进制包没有运行时依赖。官方有Docker image
* **Integrated高度集成 **官方的 Go 和 Python库都有提供。而且为大多数语言提供了库。

#### 组件

nsq 有三个必要的组建nsqd、nsqlookupd、nsqadmin 其中nsqd 和 nsqlookup是必须部署的 下面我们一一介绍。

#### **nsqd**

负责接收消息，存储队列和将消息发送给客户端，nsqd 可以多机器部署，当你使用客户端向一个topic发送消息时，可以配置多个nsqd地址，消息会随机的分配到各个nsqd上，nsqd优先把消息存储到内存channel中，当内存channel满了之后，则把消息写到磁盘文件中。他监听了两个tcp端口，一个用来服务客户端，一个用来提供http的接口 ，nsqd 启动时置顶下nsqlookupd地址即可：

```xml
nsqd --lookupd-tcp-address=127.0.0.1:4160
```

也可以指定端口 与数据目录，其他配置项可详见官网

```xml
nsqd --lookupd-tcp-address=127.0.0.1:4160 --broadcast-address=127.0.0.1 -tcp-address=127.0.0.1:4154 -http-address="0.0.0.0:4155" --data-path=/data/nsqdata
```

#### nsqlookupd

主要负责服务发现 负责nsqd的心跳、状态监测，给客户端、nsqadmin提供nsqd地址与状态。是整个集群的总控室，包括服务发现和节点拓扑信息的管理。nsqlookupd有以下特点：

1. 唯一性,在集群中的节点只能指向唯一的nsqlookupd服务
2. 去中心化,即使nsqlookupd崩溃，也会不影响正在运行的nsqd服务
3. 充当nsqd和naqadmin信息交互的中间件
4. 提供一个http查询服务，给客户端定时更新nsqd的地址目录

#### nsqadmin

nsqadmin是一个web管理界面 启动方式如下：

```xml
nsqadmin --lookupd-http-address=127.0.0.1:4161
```

#### reference

* [http://kuangjue.com/article/250](http://kuangjue.com/article/250)
* https://mp.weixin.qq.com/s/lrbIx88Z1HwWNTO\_5aABJQ



