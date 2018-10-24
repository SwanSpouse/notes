### NSQ

nsq 有三个必要的组建nsqd、nsqlookupd、nsqadmin 其中nsqd 和 nsqlookup是必须部署的 下面我们一一介绍。

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

主要负责服务发现 负责nsqd的心跳、状态监测，给客户端、nsqadmin提供nsqd地址与状态

#### nsqadmin

nsqadmin是一个web管理界面 启动方式如下：

```xml
nsqadmin --lookupd-http-address=127.0.0.1:4161
```

#### reference 

* http://kuangjue.com/article/250



