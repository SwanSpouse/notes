### gorountine:

#### gorountine概念

goroutine就是一段代码，一个函数入口，以及在堆上为其分配的一个堆栈。所以它非常廉价，我们可以很轻松的创建上万个goroutine。

goroutine是协作式调度的，如果goroutine会执行很长时间，而且不是通过等待读取或写入channel的数据来同步的话，就需要主动调用Gosched\(\)来让出CPU。

Blocking is fine : If a goroutine blocks on system call, it blocks it’s running thread. But another thread is taken from the waiting queue of Scheduler \(the Sched struct\) and used for other runnable goroutines.

The go runtime scheduler does cooperative scheduling, which means another goroutine will only be scheduled if the current one is blocking or done. Some of these cases are:

* Channel send and receive operations, if those operations would block.
* The Go statement, although there is no guarantee that new goroutine will be scheduled immediately.
* Blocking syscalls like file and network operations.
* After being stopped for a garbage collection cycle.

This is better than pre-emptive scheduling which uses timely system interrupts \(e.g. every 10 ms\) to block and schedule a new thread which may lead a task to take longer than needed to finish when number of threads increases or when a higher priority tasks need to be scheduled while a lower priority task is running.

goroutine最大的价值是其实现了并发协程和实际并行执行的线程的映射以及动态扩展，随着其运行库的不断发展和完善，其性能一定会越来越好，尤其是在CPU核数越来越多的未来，终有一天我们会为了代码的简洁和可维护性而放弃那一点点性能的差别。

#### gorountine的运作过程

封装main函数的Gorountine是Go语言runtime system创建的第一个Goroutine（主Goroutine）。

主Goroutine是在runtime.m0上被运行的。封装了引导程序的runtime.go就是在runtime.m0被运行的。实际上，在runtime.m0在运行完runtime.g0中的引导程序之后，会接着运行主Goroutine。

主Goroutine所做的事情并不是执行main函数那么简单。它首先要做的，是设定一个Goroutine所能申请的栈空间的最大尺寸。在32位的计算机系统下，这个最大尺寸是250MB，而在64位的计算机系统中，此尺寸为1GB。如果有某个Goroutine申请的栈空间总尺寸大于了这个限制，那么runtime system就会发起一个“栈溢出 stack overflow”的panic。

在设定好Goroutine的最大栈尺寸之后，主Goroutine会启动系统监测器。系统监测器的作用就是对调度器的工作进行查缺补漏。这也是让系统监测器先于main函数的执行原因之一。

此后，主Goroutine会进行一些列的初始化工作。由于这些工作的重要性和特殊性，主Goroutine会在此期间与当前M（即runtime.m0）锁定在一起。

#### 主Goroutine的执行过程

创建一个特殊的defer语句，以执行主Goroutine退出时必要的善后工作。实际上，这里的善后处理即是指主Goroutine与当前M的解锁操作。因为，主Goroutine也可能会非正常地结束，所以这一点很有必要。

检查当前M是否是runtime.m0。如果不是，那么就说明之前的程序出现了某种问题。这时，主Goroutine会立即抛出异常。这也意味着Go程序启动的失败。

执行main包的init函数。

对之前创建的那个特殊的defer语句进行最后的检查和设置，并在必要时抛出异常。如果上述初始化工作成功完成，那么主Goroutine就会去执行main函数。在执行完main函数之后，它还会检查是否有Goroutine发生了panic，并进行必要的处理。最后，主Goroutine会结束自己以及当前线程的运行。

#### runtime包与Goroutine

runtime.GOMAXPROCS函数

* 通过调用runtime.GOMAXPROCS函数，应用程序可以在运行期间设置runtime system中的P的最大数量。由于调用runtime.GOMAXPROCS的时候会"Stop the world"，所以应用程序应该尽早的调用它。

runtime.Goexit函数

* 函数runtime.Goexit被调用之后会立即使调用它的Gorountine的运行被终止，但其他Goroutine并不会受此影响。runtime.Goexit函数在终止调用它的Goroutine的运行之前会执行该Gorountine中所有还未被执行的defer语句。
* 该函数会把被终止运行的Gorountine置为Gdead状态，并将其放入调度器的自由G列表。这样，调度器可以在需要时重新启用此Gorountine。

runtime.Goshed函数

* 该函数的作用是暂停调用它的Gorountine的运行。调用它的Gorountine会被重新置于Grunnable状态，并被放入到调度器的可运行G队列中。

runtime.NumGorountine函数

* runtime.NumGorountine在被调用之后会返回runtime system中的处于特定状态（Grunnable， Grunning， Gsyscall和Gwaiting）Gorountine的数量。处于这些状态的Gorountine即被看作是活跃的或者说正在被调度的。

runtime.LockOSTread和runtime.UnLockOSThread

* 前者使调用它的Gorountine与当前运行它的M锁定在一起，后者是解除这样的锁定。多次调用不会产生问题。

#### reference 

* https://codeburst.io/why-goroutines-are-not-lightweight-threads-7c460c1f155f



