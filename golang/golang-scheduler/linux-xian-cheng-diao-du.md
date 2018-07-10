### Linux 线程调度

实时与非实时之分。实时就是指操作系统对一些中断等的响应时效性非常高，即使是在内核态的时候，非实时反之。目前像VxWorks属于实时操作系统，大家常用的windows,linux则属于非实时系统，也叫分时操作系统。响应实时的表现主要是抢占，抢占通过优先级来控制的，优先级高的任务最先占用CPU。

Linux系统中常用的几种调度类为SCHED\_NORMAL、SCHED\_FIFO、SCHED\_RR。

* 其中SCHED\_NORMAL是用于普通线程的调度类，
* SCHED\_FIFO和SCHED\_RR是用于实时线程的调度类，优先级高于SCHED\_NORMAL。

内核中区分普通线程与实时线程是根据线程的优先级，实时线程拥有实时优先级（real-time priority），默认取值为0~99，数值越高优先级越高，而普通线程只具有nice值，nice值映射到用户层的取值范围为-20～+19，数值越高优先级越低，默认初始值为0 ，子线程会继承父线程的优先级。

对于实时线程，Linux系统会尽量使其调度延时在一个时间期限内，但是不能保证总是如此，不过正常情况下已经可以满足比较严格的时间要求了。

**SCHED\_NORMAL**

对于SCHED\_NORMAL，2.6之前的版本内核中的调度管理程序是根据线程的优先级（nice值）来分配一个固定的时间片（timeslice）给线程，比如nice值0对应于100ms的时间片，而nice值-20对应于5ms时间片，线程在执行过程中进入阻塞状态（I/O操作等引起）或者是执行的时间达到分配的时间片后将会被抢占，而新进入运行态的线程会根据其优先级和可用时间片来决定是否抢占当前正在执行的程序。

2.6之后版本的Linux中SCHED\_NORMAL使用的是Linux 内核在2.6.23版本中引入的CFS（Complete Fair Scheduler）调度管理程序。CFS与之前的调度不同的是，线程的优先级与时间片之间并没有一个固定的关系，而是影响该线程在整个系统CPU运行时间中占有比例的一个因素。比如有两个线程，对应的nice值分别为0（普通线程）和+19（低优先级线程），那么普通线程将会占有19/20×100%的CPU时间，而低优先级线程将会占有1/20×100%的CPU时间（具体数值只做举例说明用，Linux内核中的计算出来的数值会不一样）。而如果同时运行的只有两个相同优先级的线程，那么他们分到的CPU时间各是50%。这样每个线程能够分配到的CPU时间占有比例跟系统当前的负载（所有处于运行态的线程数以及各线程的优先级）有关，同一个线程在他本身优先级不变的情况下分到的CPU时间占比会根据系统负载变化而发生变化，也即与时间片没有一个固定的对应关系。

理想情况下CFS对CPU时间占比的衡量是在一个无限小的时间片内计算单个线程执行时间的占比，CFS的目标是所有线程在这个小的时间片周期内执行的时间都是相同的，无限小在现实中显然是不存在的，Linux系统中这个时间片周期是与系统CPU数有关的，默认情况下单核CPU对应6ms，双核CPU对应12ms，8核CPU对应24ms，当线程数增加到很大数量时，CFS会保证每个线程获得最小执行时间， 单核CPU对应0.75ms，双核CPU对应1.5ms，8核CPU对应3ms。在CFS管理下，某个线程运行时如果进入阻塞态（或其他非运行态）或者当前时间片周期内的CPU时间占比达到后将会暂停运行，CFS然后将会选择当前时间片周期内已执行时间最少的运行态线程继续运行。当然CPU时间占比在内核中也是以运行时间衡量的，比如某个单核CPU系统中只有两个相同优先级的线程同时处于运行态，那么CFS将会以6ms为周期来调度所有线程，而每个6ms周期内每个线程分得3ms执行时间，如果某个线程中有I/O操作等其他操作使该线程进入非运行态，CFS将会立即使另外一个线程继续运行，如果两个线程都是基本没有I/O操作等会引起阻塞的操作（比如忙循环），那么线程将会在自己的执行时间结束（本质上是超出CPU时间占比）后被CFS程序调度出当前正在执行的状态，另外一个线程获得CPU资源开始执行。

需要注意的是，进入CFS（或其他调度算法）需要调度事件的产生，调度事件可以是线程自己调用函数显示执行调度，或者线程执行I/O操作等会进入阻塞的操作以及等待的事件发生线程进入运行态等（内核中有固定的调度点），如果一个程序一直处于忙计算（比如忙循环程序），那么就会需要系统软时间中断来产生调度事件从而进入CFS调度判断下一个可执行程序。目前我们的Linux内核普遍配置的系统软时间中断产生的频率为100Hz，也即每10ms产生一次中断，那么系统中只有忙计算类（如忙循环）线程的情况下，只有当系统产生软时间中断时，CFS才会被调用来判断下一个执行的线程并使其占有CPU开始执行，这个现象看起来就好象是Linux调度策略简单的给每个线程分配了10ms的时间片，其实并不是这样。如果系统中同时有忙计算类的线程和经常进行I/O操作类的线程，由于I/O类线程基本处于等待事件的阻塞态中，执行的时间很少，而计算类线程在执行的时间会比较长，如果计算类线程正在执行时，I/O类线程等待的事件发生了，CFS马上就会判断出I/O类线程在之前时间段内执行的时间很少，即已使用的CPU占比与分配给他的相比很小，而计算类线程很有可能已经超过了分配的CPU占比，那么CFS将会马上使I/O类的线程占有CPU开始执行，如此系统总是能及时响应I/O类线程。

**SCHED\_FIFO和SCHED\_RR**

SCHED\_FIFO和SCHED\_RR是实时线程使用的调度管理算法。

SCHED\_FIFO即先进先出，处于相同优先级的实时线程会根据进入运行态的次序依次执行。正在执行的线程会一直执行直到线程阻塞或者其主动调用调度线程放弃执行，处于此调度策略下的线程没有预先分配的时间片，可以永远执行下去。只有拥有更高实时优先级且处于SCHED\_RR或者SCHED\_FIFO管理下的线程能抢占正在运行的实时线程。

SCHED\_RR在SCHED\_FIFO的基础上会预先给定线程一个时间片，时间片达到后会使其他相同优先级的线程开始执行。SCHED\_RR的时间片轮询机制只对同等实时优先级的线程有效，更高实时优先级的线程总是会抢占正在执行的线程，而低优先级的线程不能抢占高优先级的线程，即使其时间片已到。

实时线程优先级高于所有普通线程，如果有实时线程处于运行态，则系统调度时一定会选择调用实时线程；正在运行的实时线程只会被拥有更高实时优先级的线程抢占。所以在应用中如果需要将某个线程设置为实时线程，则需要用户自己确保该线程不会处于忙执行而完全占用CPU资源，导致其他普通线程没法获得CPU资源而一直被阻塞得不到执行，并且需要合理给予优先级的值，太高有可能会影响重要系统线程的运行。**所有用户态线程默认没有实时优先级，都属于普通线程。**

#### reference

* [http://www.emtronix.com/article/article20171018.html](http://www.emtronix.com/article/article20171018.html)


