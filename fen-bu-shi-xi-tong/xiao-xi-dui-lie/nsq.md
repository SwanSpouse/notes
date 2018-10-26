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

#### Topic 

一个topic就是程序发布消息的一个逻辑键，当程序第一次发布消息时就会创建topic。

#### Channels

channel与消费者相关，是消费者之间的负载均衡，channel在某种意义上来说是一个“队列”。每当一个发布者发送一条消息到一个topic，消息会被复制到所有消费者连接的channel上，消费者通过这个特殊的channel读取消息，实际上，在消费者第一次订阅时就会创建channel。Channel会将消息进行排列，如果没有消费者读取消息，消息首先会在内存中排队，当量太大时就会被保存到磁盘中。

#### Message

消息构成了数据流的中坚力量，消费者可以选择结束消息，表明它们正在被正常处理，或者重新将他们排队待到后面再进行处理。每个消息包含传递尝试的次数，当消息传递超过一定的阀值次数时，我们应该放弃这些消息，或者作为额外消息进行处理。

### 常用工具类

* nsq\_to \_file：消费指定的话题（topic）/通道（channel），并写到文件中，有选择的滚动和/或压缩文件。

* nsq\_to \_http：消费指定的话题（topic）/通道（channel）和执行 HTTP requests \(GET/POST\) 到指定的端点。

* nsq\_to \_nsq：消费者指定的话题/通道和重发布消息到目的地 nsqd 通过 TCP。

### 拓扑结构

NSQ推荐通过他们相应的nsqd实例使用协同定位发布者，这意味着即使面对网络分区，消息也会被保存在本地，直到它们被一个消费者读取。更重要的是，发布者不必去发现其他的nsqd节点，他们总是可以向本地实例发布消息。

  
![](/assets/nsq拓扑结构.png)

首先，一个发布者向它的本地nsqd发送消息，要做到这点，首先要先打开一个连接，然后发送一个包含topic和消息主体的发布命令，在这种情况下，我们将消息发布到事件topic上以分散到我们不同的worker中。

事件topic会复制这些消息并且在每一个连接topic的channel上进行排队，在我们的案例中，有三个channel，它们其中之一作为档案channel。消费者会获取这些消息并且上传到S3。

![](/assets/nsqd.png)

每个channel的消息都会进行排队，直到一个worker把他们消费，如果此队列超出了内存限制，消息将会被写入到磁盘中。Nsqd节点首先会向nsqlookup广播他们的位置信息，一旦它们注册成功，worker将会从nsqlookup服务器节点上发现所有包含事件topic的nsqd节点。

![](/assets/nsqlookupd.png)

然后每个worker向每个nsqd主机进行订阅操作，用于表明worker已经准备好接受消息了。这里我们不需要一个完整的连通图，但我们必须要保证每个单独的nsqd实例拥有足够的消费者去消费它们的消息，否则channel会被队列堆着。

#### reference

* [http://kuangjue.com/article/250](http://kuangjue.com/article/250)
* [https://mp.weixin.qq.com/s/lrbIx88Z1HwWNTO\_5aABJQ](https://mp.weixin.qq.com/s/lrbIx88Z1HwWNTO_5aABJQ)



