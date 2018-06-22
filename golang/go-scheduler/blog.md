
## The Go scheduler

### 简介：

* Dmitry Vyukov在Go1.1版本中对go scheduler（调度器）进行了重大升级。新的scheduler极大的提升了go程序并发执行的效率。本文就是对新版scheduler进行简单的介绍。

* 这篇文章所说的内容都可以在[设计文档](https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit)中找到。但是这篇文档过于学术，不太容易读懂。

* All you need to know about the new scheduler is in that design document but this post has pictures, so it's clearly superior.

### 带有调度器的Go运行时系统（golang runtime）都需要些什么？

* 在我们开始研究新调度器之前，我们应该明白为什么需要这样一个调度器。 既然操作系统在内核空间(kernal space)已经提供了一个线程调度器，为什么还要在用户空间(user space)创建一个新的调度器？

* The POSIX thread API is very much a logical extension to the existing Unix process model and as such, threads get a lot of the same controls as processes. Threads have their own signal mask, can be assigned CPU affinity, can be put into cgroups and can be queried for which resources they use. All these controls add overhead for features that are simply not needed for how Go programs use goroutines and they quickly add up when you have 100,000 threads in your program.

* 对于Unix等进程模型而言，使用POSIX (可移植操作系统接口, Portable Operating System Interface of UNIX，缩写为 POSIX ）API 是非常符合逻辑的。因为线程拥有自己的信号掩码,

* 另一个问题是：系统无法通知线程系统即将要进行线程调度。例如，在go语言在进行垃圾回收时，需要所有线程停止运行并且内存处于稳定的状态。这就要等待所有线程都到达内存稳定的状态（安全点 safe point）。

* 当有很多线程在运行的时候，它们内存的状态“杂乱无章”(没有处于稳定的状态)。所以通常情况下不得不等待所有线程都到达内存稳定的状态。然后在go的线程模型中，调度器可以保证在线程运行到内存稳定状态的时候再切换线程。这意味着，当我们需要进行垃圾回收的时候，我们只需要等待那些真正在CPU上运行的那些线程达到内存稳定的状态。（因为在线程切换的时候，线程内存必须到达稳定状态，才能够切换到下一个线程。）


### Our Cast of Characters 线程模型

* 有3种常见的线程模型。第一种是N:1（任意个用户线程对应1个系统线程），这种线程模型的优点就是可以快速的对现场进行切换，但是在多核系统中，不能够对CPU资源进行充分利用； 第二种是1:1(1个用户线程对应1个系统线程)，这种方式的优点就是可以充分利用系统的CPU资源，但缺点就是线程切换需要花费更多的时间。

* go综合了两种方式的优点，采用了M:N的线程模型（任意个用户线程对应任意个系统线程）。这种方式的优点在于可以快速地进行线程切换且充分利用了系统CPU资源。但这种方式的难点在于调度器的实现。

	![entities](http://images.cnblogs.com/cnblogs_com/swanspouse/1159380/o_go1.jpg)

* 为了实现调度的任务。go调度器使用了3个主要的实体:

    * 矩形代表了由操作系统管理执行的系统线程。它工作的方式和标准的POSIX线程很像。在go 运行时系统中，被称为M (Machine)

    * 圆形代表了gorountine，它拥有自己的栈内存(stack), 指令指针 (instruction pointer) 和用于调度的一些重要信息。在go运行时系统中，被称为G（gorountine)

    * 三角形代表用于调度的上下文环境。可以把它看做是一个本地化的调度器，在这个调度器上可以启动一个go线程来运行go代码。它是实现N:1调度器到M:N调度器的重要一步。在go运行时系统中，被称为P(processor)。

	![goo2](http://images.cnblogs.com/cnblogs_com/swanspouse/1159380/o_go2.jpg)

* 上图中有两个线程（M），每个线程拥有一个上下文环境（P），每个上线文环境中运行着一个Goroutine（G）。为了能够按顺序执行Goroutine， 每一个线程必须独占一个上下文环境（P）。

* 其中，线程运行上下文环境(P)的数量是在进程启动的时候通过GOMAXPROCS环境变量或者通过运行时系统函数GOMAXPROCS()来设定。通常情况下，在程序运行期间这个值是不能被改变的，也就是说在任意时刻，运行go代码的上下文环境的数量是固定的。根据不同的计算资源，我们可以设定不同的上下文环境（P）的数量。例如，在一个4核的个人电脑上，运行4个GO 上下文环境。(4个线程)

* 上图中，灰色的Gorountines是当前没有在运行的，但是正处于就绪状态（随时可以开始运行）。它们被放置在一个叫做 runqueues 的列表中。当任何在代码中显示执行go语句的时候，一个Goroutine就会被添加到这个列表的末尾。一旦当前正在运行的Goroutine运行到一个可以调度的点之后（safe point），它从会从runqueue中取出一个等待的gorountine，设置其栈内存，构造函数指针等然后开始执行就绪的gorountine。

* 为降低互斥竞争的问题，每一个上下文环境都拥有自己的runqueue。在上一个版本的go调度器中，只有一个带锁的全局runqueue。因此，线程经常会因为不能同时访问临界区资源而等待。所以，当在32核的机器上运行时，线程运行的性能会明显下降。

* 只要在runqueue中拥有等待的gorountine，调度器就会保持这样的状态稳定的运行。然而，有很多情况会打破这一平衡。

### Who you gonna (sys)call?

* You might wonder now, why have contexts at all? Can't we just put the runqueues on the threads and get rid of contexts? Not really. The reason we have contexts is so that we can hand them off to other threads if the running thread needs to block for some reason.

* 到这可能会有疑问，为什么需要上下文环境？为什么不直接把runqueues放到线程之中。其原因是希望当正在运行的线程需要阻塞的时候，runqueue、调度器不被阻塞，其它线程可以继续运行。

* An example of when we need to block, is when we call into a syscall. Since a thread cannot both be executing code and be blocked on a syscall, we need to hand off the context so it can keep scheduling.

* 例如：在我们希望线程进行阻塞的时候，我们会进行一个系统调用。但线程不能在运行代码的同时被系统调用所阻塞，而这时，我们需要对上下文环境进行一些处理，保证它一直在进行着调度而没有被阻塞。

	![go3](http://images.cnblogs.com/cnblogs_com/swanspouse/1159380/o_go3.jpg)

* Here we see a thread giving up its context so that another thread can run it. The scheduler makes sure there are enough threads to run all contexts. M1 in the illustration above might be created just for the purpose of handling this syscall or it could come from a thread cache. The syscalling thread will hold on to the goroutine that made the syscall since it's technically still executing, albeit blocked in the OS.

* 上图中，可以看到。当一个线程要交出其运行的上下文环境，供其它线程运行时，`调度器保证每个上下文环境都有足够的线程在运行。图中M1可能是专门用来处理系统调用或者来是缓存线程。` 

When the syscall returns, the thread must try and get a context in order to run the returning goroutine. The normal mode of operation is to steal a context from one of the other threads. If it can't steal one, it will put the goroutine on a global runqueue, put itself on the thread cache and go to sleep.

* 当系统调用返回时，线程必须尝试去获取一个上下文环境以便能够继续运行。通常情况下做法是从其他线程那里偷取一个上下文环境。如果它没能够偷取到上下文环境，它就会被放到一个全局的runqueue之中，然后进行睡眠。

The global runqueue is a runqueue that contexts pull from when they run out of their local runqueue. Contexts also periodically check the global runqueue for goroutines. Otherwise the goroutines on global runqueue could end up never running because of starvation.

* 当上下文环境发现本地的runqueue之中没有可一直运行的gorountine了，就会从全局的runqueue之中拉取gorountine。上下文环境会对全局的runqueue进行定时的检查，否则全局runqueue中的gorountine就会因为永远得不到运行的机会而饿死。

This handling of syscalls is why Go programs run with multiple threads, even when GOMAXPROCS is 1. The runtime uses goroutines that call syscalls, leaving threads behind.

* 通过这种做法

### Stealing work "偷取" goroutine

* Another way that the steady state of the system can change is when a context runs out of goroutines to schedule to. This can happen if the amount of work on the contexts' runqueues is unbalanced. This can cause a context to end up exhausting it's runqueue while there is still work to be done in the system. To keep running Go code, a context can take goroutines out of the global runqueue but if there are no goroutines in it, it'll have to get them from somewhere else.

* 还有一种可以打破系统稳定状态的情况是当一个上下文环境没有可以继续调度的gorountine的时候。这种情况通常发生在上下文环境本地runqueue gorountine任务不均衡导致的。这会使得其中某些上下文环境一直在忙碌着执行任务，而另一些上下文环境没有任务可做。

	![go4](http://images.cnblogs.com/cnblogs_com/swanspouse/1159380/o_go4.jpg)
	
That somewhere is the other contexts. When a context runs out, it will try to steal about half of the runqueue from another context. This makes sure there is always work to do on each of the contexts, which in turn makes sure that all threads are working at their maximum capacity.

* 这时，无事可做的上下文环境会尝试着从另一个上下文环境的runqueue中偷一半儿的任务回来。这保证了在任何时候，所有的上下文环境都有任务可做，也使得线程提高了处理任务的能力。

### Where to go?
* There are many more details to the scheduler, like cgo threads, the LockOSThread() function and integration with the network poller. These are outside the scope of this post, but still merit study. I might write about these later. There are certainly plenty of interesting constructions to be found in the Go runtime library.

* 关于调度器还有很多的细节，例如cgo线程，LockOSThread()函数、和网络轮询的整合等。这就在这篇博客的内容之外了，但仍然非常值得学习。我会在以后写一些关于上述问题的文章。在go runtime包中，有很多非常值得我们学习，也非常有趣的设计和构造。

### 参考： 
* http://morsmachine.dk/go-scheduler
* https://studygolang.com/articles/7852


