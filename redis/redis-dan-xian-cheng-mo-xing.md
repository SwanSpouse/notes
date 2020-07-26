# redis 单线程模型

无论是使用单线程模型还是多线程模型，这两个设计上的决定都是为了更好地提升 Redis 的开发效率、运行性能，想要理解两个看起来矛盾的设计决策，我们首先需要重新梳理做出决定的上下文和大前提，从下面的角度来看，使用单线程模型和多线程模型其实也并不矛盾。

虽然 Redis 在较新的版本中引入了多线程，不过是在**部分命令**上引入的，其中包括非阻塞的删除操作，在整体的架构设计上，主处理程序还是单线程模型的；由此看来，我们今天想要分析的两个问题可以简化成：

* 为什么 Redis 服务使用单线程模型处理绝大多数的网络请求？
* 为什么 Redis 服务增加了多个非阻塞的删除操作，例如：`UNLINK`、`FLUSHALL ASYNC`和`FLUSHDB ASYNC`？

接下来的两个小节将从多个角度分析这两个问题。

## 单线程模型

Redis 从一开始就选择使用单线程模型处理来自客户端的绝大多数网络请求，这种考虑其实是多方面的，作者分析了相关的资料，发现其中最重要的几个原因如下：

1. 使用单线程模型能带来更好的可维护性，方便开发和调试；
2. 使用单线程模型也能并发的处理客户端的请求；
3. Redis 服务中运行的绝大多数操作的性能瓶颈都不是 CPU；

上述**三个原因中的最后一个是最终使用单线程模型的决定性因素，**其他的两个原因都是使用单线程模型额外带来的好处，在这里我们会按顺序介绍上述的几个原因。

#### 并发处理 {#并发处理}

使用单线程模型也并不意味着程序不能并发的处理任务，Redis 虽然使用单线程模型处理用户的请求，但是它却使用 I/O 多路复用机制**并发**处理来自客户端的多个连接，同时等待多个连接发送的请求。

在 I/O 多路复用模型中，最重要的函数调用就是`select`以及类似函数，该方法的能够同时监控多个文件描述符（也就是客户端的连接）的可读可写情况，当其中的某些文件描述符可读或者可写时，`select`方法就会返回可读以及可写的文件描述符个数。

使用 I/O 多路复用技术能够极大地减少系统的开销，系统不再需要额外创建和维护进程和线程来监听来自客户端的大量连接，减少了服务器的开发成本和维护成本。

#### 性能瓶颈

最后要介绍的其实就是 Redis 选择单线程模型的决定性原因 —— 多线程技术能够帮助我们充分利用 CPU 的计算资源来并发的执行不同的任务，但是 CPU 资源往往都不是 Redis 服务器的性能瓶颈。哪怕我们在一个普通的 Linux 服务器上启动 Redis 服务，它也能在 1s 的时间内处理 1,000,000 个用户请求。

It’s not very frequent that CPU becomes your bottleneck with Redis, as usually Redis is either memory or network bound. For instance, using pipelining Redis running on an average Linux system can deliver even 1 million requests per second, so if your application mainly uses O\(N\) or O\(log\(N\)\) commands, it is hardly going to use too much CPU.

如果这种吞吐量不能满足我们的需求，更推荐的做法是使用分片的方式将不同的请求交给不同的 Redis 服务器来处理，而不是在同一个 Redis 服务中引入大量的多线程操作。

简单总结一下，Redis 并不是 CPU 密集型的服务，如果不开启 AOF 备份，所有 Redis 的操作都会在内存中完成不会涉及任何的 I/O 操作，这些数据的读写由于只发生在内存中，所以处理速度是非常快的；整个服务的瓶颈在于网络传输带来的延迟和等待客户端的数据传输，也就是网络 I/O，所以使用多线程模型处理全部的外部请求可能不是一个好的方案。

多线程虽然会帮助我们更充分地利用 CPU 资源，但是操作系统上线程的切换也不是免费的，线程切换其实会带来额外的开销，其中包括：

1. 保存线程 1 的执行上下文；
2. 加载线程 2 的执行上下文；

频繁的对线程的上下文进行切换可能还会导致性能地急剧下降，这可能会导致我们不仅没有提升请求处理的平均速度，反而进行了负优化，所以这也是为什么 Redis 对于使用多线程技术非常谨慎。

## 引入多线程

Redis 在最新的几个版本中加入了一些可以被其他线程异步处理的删除操作，也就是我们在上面提到的`UNLINK`、`FLUSHALL ASYNC`和`FLUSHDB ASYNC`，我们为什么会需要这些删除操作，而它们为什么需要通过多线程的方式异步处理？

#### 删除操作

我们可以在 Redis 在中使用`DEL`命令来删除一个键对应的值，如果待删除的键值对占用了较小的内存空间，那么哪怕是**同步地**删除这些键值对也不会消耗太多的时间。

但是对于 Redis 中的一些超大键值对，几十 MB 或者几百 MB 的数据并不能在几毫秒的时间内处理完，Redis 可能会需要在释放内存空间上消耗较多的时间，这些操作就会阻塞待处理的任务，影响 Redis 服务处理请求的 PCT99 和可用性。

然而释放内存空间的工作其实可以由后台线程异步进行处理，这也就是`UNLINK`命令的实现原理，它只会将键从元数据中删除，真正的删除操作会在后台异步执行。

### 总结

Redis 选择使用单线程模型处理客户端的请求主要还是因为 CPU 不是 Redis 服务器的瓶颈，所以使用多线程模型带来的性能提升并不能抵消它带来的开发成本和维护成本，系统的性能瓶颈也主要在网络 I/O 操作上；而 Redis 引入多线程操作也是出于性能上的考虑，对于一些大键值对的删除操作，通过多线程非阻塞地释放内存空间也能减少对 Redis 主线程阻塞的时间，提高执行的效率。

### 参考

* [https://draveness.me/whys-the-design-redis-single-thread/](https://draveness.me/whys-the-design-redis-single-thread/)


