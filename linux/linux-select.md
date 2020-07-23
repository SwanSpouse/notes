# Linux  select

select系统调用的的用途是：在一段指定的时间内，监听用户感兴趣的文件描述符上可读、可写和异常等事件。

```
int iResult = recv(s, buffer,1024);
```

这是用来接收数据的，在默认的阻塞模式下的套接字里，recv会阻塞在那里，直到套接字连接上有数据可读，把数据读到buffer里后recv函数才会返回，不然就会一直阻塞在那里。在单线程的程序里出现这种情况会导致主线程（单线程程序里只有一个默认的主线程）被阻塞,这样整个程序被锁死在这里，如果永远没数据发送过来，那么程序就会被永远锁死。这个问题可以用多线程解决，但是在有多个套接字连接的情况下，这不是一个好的选择，扩展性很差。

```
int iResult = ioctlsocket(s, FIOBIO, (unsigned long *)&ul);
iResult = recv(s, buffer,1024);
```

这一次recv的调用不管套接字连接上有没有数据可以接收都会马上返回。原因就在于我们用ioctlsocket把套接字设置为非阻塞模式了。不过你跟踪一下就会发现，在没有数据的情况下，recv确实是马上返回了，但是也返回了一个错误：**WSAEWOULDBLOCK**，意思就是请求的操作没有成功完成。

看到这里很多人可能会说，那么就重复调用recv并检查返回值，直到成功为止，但是这样做效率很成问题，开销太大。

select模型的出现就是为了解决上述问题。  
select模型的关键是使用一种有序的方式，对多个套接字进行统一管理与调度 。![](/assets/linux select.png)

如上所示，用户首先将需要进行IO操作的socket添加到select中，然后阻塞等待select系统调用返回。当数据到达时，socket被激活，select函数返回。用户线程正式发起read请求，读取数据并继续执行。

  
从流程上来看，使用select函数进行IO请求和同步阻塞模型没有太大的区别，甚至还多了添加监视socket，以及调用select函数的额外操作，效率更差。**但是，使用select以后最大的优势是用户可以在一个线程内同时处理多个socket的IO请求。**用户可以注册多个socket，然后不断地调用select读取被激活的socket，即可达到在同一个线程内同时处理多个IO请求的目的。而在同步阻塞模型中，必须通过多线程的方式才能达到这个目的。

```
{
	select(socket);
	while(1) 
	{
		sockets = select();
		for(socket in sockets) 
		{
			if(can_read(socket)) 
			{
				read(socket, buffer);
				process(buffer);
			}
		}
	}
}

```

## select相关API介绍与使用

```
#include <sys/select.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
int select(int maxfdp, fd_set *readset, fd_set *writeset, fd_set *exceptset,struct timeval *timeout);
```

参数说明：

maxfdp：被监听的文件描述符的总数，它比所有文件描述符集合中的文件描述符的最大值大1，因为文件描述符是从0开始计数的；

readfds、writefds、exceptset：分别指向可读、可写和异常等事件对应的描述符集合。

timeout:用于设置select函数的超时时间，即告诉内核select等待多长时间之后就放弃等待。timeout == NULL 表示等待无限长的时间





















