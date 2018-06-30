### 系统调用

除了内存管理、文件管理、进程管理、外设管理等等内部模块以外，操作系统还提供了许多外部接口供应用程序使用，这些接口就是所谓的“系统调用”。

从DOS时代开始，系统调用就是通过软中断的形式来提供，也就是著名的INT 21，程序把需要调用的功能编号放入AH寄存器，把参数放入其他指定的寄存器，然后调用INT 21，中断返回后，程序从指定的寄存器\(通常是AL\)里取得返回值。这样的做法一直到奔腾2也就是P6出来之前都没有变，譬如windows通过INT 2E提供系统调用，Linux则是INT 80，只不过后来的寄存器比以前大一些，而且可能再多一层跳转表查询。后来，Intel和AMD分别提供了效率更高的SYSENTER/SYSEXIT和SYSCALL/SYSRET指令来代替之前的中断方式，略过了耗时的特权级别检查以及寄存器压栈出栈的操作，直接完成从RING 3代码段到RING 0代码段的转换。

系统调用都提供什么功能呢？用操作系统的名字加上对应的中断编号到谷歌上一查就可以得到完整的列表 \(Windows, Linux\)，这个列表就是操作系统和应用程序之间沟通的协议，如果需要超出此协议的功能，我们就只能在自己的代码里去实现，譬如，对于内存管理，操作系统只提供进程级别的内存段的管理，譬如Windows的virtualmemory系列，或是Linux的brk，操作系统不会去在乎应用程序如何为新建对象分配内存，或是如何做垃圾回收，这些都需要应用程序自己去实现。如果超出此协议的功能无法自己实现，那我们就说该操作系统不支持该功能，举个例子，Linux在2.6之前是不支持多线程的，无论如何在程序里模拟，我们都无法做出多个可以同时运行的并符合POSIX 1003.1c语义标准的调度单元。

可是，我们写程序并不需要去调用中断或是SYSCALL指令，这是因为操作系统提供了一层封装，在Windows上，它是NTDLL.DLL，也就是常说的Native API，我们不但不需要去直接调用INT 2E或SYSCALL，准确的说，我们不能直接去调用INT 2E或SYSCALL，因为Windows并没有公开其调用规范，直接使用INT 2E或SYSCALL无法保证未来的兼容性。在Linux上则没有这个问题，系统调用的列表都是公开的，而且Linus非常看重兼容性，不会去做任何更改，glibc里甚至专门提供了syscall\(2\)来方便用户直接用编号调用，不过，为了解决glibc和内核之间不同版本兼容性带来的麻烦，以及为了提高某些调用的效率\(譬如_\_NR_ gettimeofday\)，Linux上还是对部分系统调用做了一层封装，就是VDSO \(早期叫linux-gate.so\)。

可是，我们写程序也很少直接调用NTDLL或者VDSO，而是通过更上一层的封装，这一层处理了参数准备和返回值格式转换、以及出错处理和错误代码转换，这就是我们所使用语言的运行库，对于C语言，Linux上是glibc，Windows上是kernel32\(或调用msvcrt\)，对于其他语言，譬如Java，则是JRE，这些“其他语言”的运行库通常最终还是调用glibc或kernel32。

### gorountine

#### 协程

协程和线程的原理是一样的，当a线程切换到b线程的时候，需要将a线程的相关执行进度压入栈，然后将b线程的执行进度出栈，进入b的执行序列。协程只不过是在应用层实现这一点。

但是，协程并不是由操作系统调度的，而且应用程序也没有能力和权限执行cpu调度。怎么解决这个问题？

答案是，协程是基于线程的。内部实现上，维护了一组数据结构和n个线程，真正的执行还是线程，协程执行的代码被扔进一个待执行队列中，有这n个线程从队列中拉出来执行。这就解决了协程的执行问题。那么协程是怎么切换的呢？答案是:golang对各种io函数进行了封装，这些封装的函数提供给应用程序使用，而其内部调用了操作系统的异步io函数，当这些异步函数返回busy或bloking时，golang利用这个时机将现有的执行序列压栈，让线程去拉另外一个协程的代码来执行，基本原理就是这样，利用并封装了操作系统的异步函数。包括linux的epoll，select和windows的iocp,event等。

golang的协程是目前各类有协程概念的语言中实现的最完整和成熟的。

#### gorountine:

goroutine就是一段代码，一个函数入口，以及在堆上为其分配的一个堆栈。所以它非常廉价，我们可以很轻松的创建上万个goroutine。

goroutine是协作式调度的，如果goroutine会执行很长时间，而且不是通过等待读取或写入channel的数据来同步的话，就需要主动调用Gosched\(\)来让出CPU。

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

创建定时垃圾回收器\(scavenger\)，主Goroutine会创建一个专门的Goroutine来封装这个定时垃圾回收器，并把它放入到当前M的可运行队列G队列中。注意，此时Goroutine即是Go语言runtime system创建的第二个Goroutine。创建它的方式与我们使用的go语句创建一个用户级别的Goroutine的方式几乎无二。顺便提一下，定时垃圾收集器会定时（当前是2分钟一次）的执行垃圾回收任务，并在必要时促使一些M协助它进行一些垃圾回收工作。

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

### 参考

* 《Go并发编程实战》
* [http://www.sizeofvoid.net/goroutine-under-the-hood/](http://www.sizeofvoid.net/goroutine-under-the-hood/)



