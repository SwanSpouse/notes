#### select 子句

select语句：仅能用于发送和接受通道中的元素值的专用语句。

golang 的 select 就是监听 IO 操作，当 IO 操作发生时，触发相应的动作。  
在执行select语句的时候，运行时系统会自上而下地判断每个case中的发送或接收操作是否可以被立即执行【立即执行：意思是当前Goroutine不会因此操作而被阻塞，还需要依据通道的具体特性\(缓存或非缓存\)】

* 每个case语句里必须是一个IO操作

* 所有channel表达式都会被求值、所有被发送的表达式都会被求值

* 如果任意某个case可以进行，它就执行\(其他被忽略\)。

* 如果有多个case都可以运行，select会随机公平地选出一个执行\(其他不会执行\)。

* 如果有default子句，case不满足条件时执行该语句。

* 如果没有default字句，select将阻塞，直到某个case可以运行；Go不会重新对channel或值进行求值。

#### 应用场景

这里说一个在项目中用到的简单应用场景：服务器要控制同一时间进行连接的客户端数量。所以初始化了一个chan并设置了最大的连接数，假如为100。

当有客户端进行连接时，首先会向conChan中写入数据，连接断开的时候，就会消费chan中的数据。

当客户端链接的数量超过100的时候，conChan会发生写入阻塞，所以会执行default操作，产生连接数过多的错误，阻止客户端和服务器进行通信。

```
conChan: make(chan bool, config.Instance().GetMaxConnection())

select {
case conChan <- true:
default:
    //当前连接数过多，请稍候再试
    return
}

defer func() {
    select {
    case <- conChan:
    default:
        // 应该不会到达这里
    }
}
```

#### 参考

* [https://blog.csdn.net/liuxinmingcode/article/details/49507991](https://blog.csdn.net/liuxinmingcode/article/details/49507991)



