# Socket

## Socket

Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。在设计模式中，Socket其实就是一个门面模式，它把复杂的

TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

![](../.gitbook/assets/socket.jpg)

服务器端先初始化Socket，然后与端口绑定\(bind\)，对端口进行监听\(listen\)，调用accept阻塞，等待客户端连接。在这时如果有个客户端初始化一个Socket，然后连接服务器\(connect\)，如果连接成功，这时客户端与服务器端的连接就建立了。客户端发送数据请求，服务器端接收请求并处理请求，然后把回应数据发送给客户端，客户端读取数据，最后关闭连接，一次交互结束。

## reference

* [https://www.cnblogs.com/wangcq/p/3520400.html](https://www.cnblogs.com/wangcq/p/3520400.html)

