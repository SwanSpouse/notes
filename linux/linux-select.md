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

返回值：超时返回0;失败返回-1；成功返回大于0的整数，这个整数表示就绪描述符的数目。

#### 以下介绍与select函数相关的常见的几个宏：

```
int FD_ZERO(int fd, fd_set *fdset);   // 一个 fd_set类型变量的所有位都设为 0
int FD_CLR(int fd, fd_set *fdset);    // 清除某个位时可以使用
int FD_SET(int fd, fd_set *fd_set);   // 设置变量的某个位置位
int FD_ISSET(int fd, fd_set *fdset);  // 测试某个位是否被置位
```

#### select使用范例：

当声明了一个文件描述符集后，必须用FD\_ZERO将所有位置零。之后将我们所感兴趣的描述符所对应的位置位，操作如下：

```
fd_set rset;   
int fd;   
FD_ZERO(&rset);   
FD_SET(fd, &rset);   
FD_SET(stdin, &rset);
```

然后调用select函数，拥塞等待文件描述符事件的到来；如果超过设定的时间，则不再等待，继续往下执行。

```
select(fd+1, &rset, NULL, NULL, NULL);
```

select返回后，用FD\_ISSET测试给定位是否置位：

```
if(FD_ISSET(fd, &rset)   
{ 
    ... 
    //do something  
}
```

简单的使用select的🌰

```
#include <sys/select.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main()
{
    fd_set rd;
    struct timeval tv;
    int err;


    FD_ZERO(&rd);
    FD_SET(0,&rd);

    tv.tv_sec = 5;
    tv.tv_usec = 0;
    err = select(1,&rd,NULL,NULL,&tv);

    if(err == 0) //超时
    {
        printf("select time out!\n");
    }
    else if(err == -1)  //失败
    {
        printf("fail to select!\n");
    }
    else  //成功
    {
        printf("data is available!\n");
    }


    return 0;
}
```

这上面的 FD\_SET\(O, &rd\) 没有太理解是什么意思；

### 深入理解select 模型

理解select模型的关键在于理解fd\_set,为说明方便，取fd\_set长度为1字节，fd\_set中的每一bit可以对应一个文件描述符fd。则1字节长的fd\_set最大可以对应8个fd。

（1）执行fd\_set set; FD\_ZERO\(&set\); 则set用位表示是0000,0000。

（2）若fd＝5,执行FD\_SET\(fd,&set\);后set变为0001,0000\(第5位置为1\)

（3）若再加入fd＝2，fd=1,则set变为0001,0011

（4）执行select\(6,&set,0,0,0\)阻塞等待

（5）若fd=1,fd=2上都发生可读事件，则select返回，此时set变为0000,0011。**注意：没有事件发生的fd=5被清空。**

基于上面的讨论，可以轻松得出select模型的特点：

（1）可监控的文件描述符个数取决与sizeof\(fd\_set\)的值。我这边服务器上sizeof\(fd\_set\)＝512，每bit表示一个文件描述符，则我服务器上支持的最大文件描述符是512\*8=4096。据说可调，另有说虽然可调，**但调整上限受于编译内核时的变量值。**

（2）将fd加入select监控集的同时，还要再使用一个数据结构array保存放到select监控集中的fd，一是用于再select返回后，array作为源数据和fd\_set进行FD\_ISSET判断。二是select返回后会把以前加入的但并无事件发生的fd清空，则每次开始select前都要重新从array取得fd逐一加入（FD\_ZERO最先），扫描array的同时取得fd最大值maxfd，用于select的第一个参数。

（3）可见select模型必须在select前循环加fd，取maxfd，**select返回后利用FD\_ISSET判断是否有事件发生。**

**理解：描述一下select使用的这个流程**

* 假设我要监听1、2、5 三个文件描述符的xx事件；分别调用 FD\_SET set进去；

* 执行select 阻塞等待；

* 当fd1 fd2 fd5其中有一个发生监听的事件发生的时候，则select 返回；这时候需要注意的是没有发生的事件会被清空。例如fd1 fd2发生了，fd5没有发生。那么fd\_set返回的就是0000,0011，1、2分别对应着上面的位置。上面说需要一个array进行记录的意思就是，如果我不记录着我注册了多少fd，那么当select返回的时候，我就丢失了未发生的事件。所以需要记录fd1 fd2 fd5，在处理事件之后，重新向select进行注册。



