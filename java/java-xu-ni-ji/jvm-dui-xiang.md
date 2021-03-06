# jvm 对象

## JVM 对象

#### 对象的创建

* 虚拟机遇到一条new指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已经被加载，解析和初始化过。如果没有，那必须先执行相应的类夹在过程。
* 在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需内存的大小在类加载完成后便可完全确定，为对象分配空间的任务等同于把一块确定大小的内存从Java堆中划分出来。

#### 内存分配的两种方法：

指针碰撞\(Bump the Pointer\) ： 假设Java堆内存是绝对规整的，所有用过的内存都放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲那边挪动一段与对象大小相等的距离。

空闲列表\(Free List\) : 如果Java堆内存并不是规整的。已使用的内存和空闲的内存相互交错，就没有办法使用指针碰撞了。虚拟机就必须维护一个列表，记录上哪些内存块是可以使用的，在分配的时候从列表中找到一个块足够大的空间划分给对象实例，并更新列表上的记录。

ps: 选择哪种分配方式是由Java堆是否规整决定的，Java堆是否规整又是由所采用的垃圾收集器是否带有压缩整理功能决定的。

* Serial、ParNew等带Compact过程的收集器，系统采用的分配方式是指针碰撞。
* CMS这种机遇Mark-Sweep算法的收集器时，通常采用空闲列表。

#### 对象的线程安全问题

解决内存分配的线程安全问题有两种方法：

* 一种是对分配的内存空间的动作进行同步处理 —— 实际上虚拟机采用CAS配上失败重试的方法保证更新操作的原子性。
* 另一种是把内存分配的动作按照线程划分到不同的空间之中进行。即每个线程在Java堆中预先分配一小块内存，称为本地线程分配缓冲（Thread Local Allocation Buffer, TLAB）。哪个线程分配内存，就在哪个线程的TLAB上分配，只有TLAB用完并分配新的TLAB时，才需要同步锁定。虚拟机是否使用TLAB，可以用过 -XX:+/-UseTLAB 参数来设定。

#### 虚拟机对对象进行必要的设置

对象属于哪个实例、如何才能找到类的元数据信息、对象的哈希码、对象的GC分代年龄等信息。这些信息存放在对象头\(Object Header\)之中。根据虚拟机的当前的运行状态的不同，如是否使用偏向锁等，对象头会有不同的设置方式。

#### 对象的内存布局

在HotSpot虚拟机中，对象在内存中存储的布局可以分为3块区域：对象头（Header）、实例数据（Instance Data）和对齐补充（Padding）。

对象头，HotSpot虚拟机的对象头包括两部分信息

* 第一部分用于存储对象自身的运行时数据，如哈希码（HashCode），GC分代年龄、锁状态标志、线程持有的错、偏向线程ID、偏向时间戳等，这部分数据在长度在32位和64位的虚拟机中分别为32bit和64bit，官方称之为“Mark Word”。
* 另一部分是类型指针。对象指向它的类元数据的指针，用于确定对象是哪个类的实例。但并不是所有对象都保留类型指针，如数组。对象头中必须还有一块用于记录数组长度的数据。

实例数据，真正存储的有效信息。也是在程序代码中定义的各种类型的字段内容。无论是从父类继承下来的，还是在子类中定义的。在分配策略中，相同宽度的字段总是被分配到一起。

对齐字段，对象的大小必须为8字节的整数倍。对象实例数据部分没有对齐时，就需要通过对齐补充来补全。

#### 对象的访问定位

Java需要通过栈上的reference数据来操作堆上的具体对象。访问方式有使用句柄和直接指针两种。

* 使用句柄访问：Java堆中会划分出一块内存来作为句柄池，reference中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息。

![](http://img.my.csdn.net/uploads/201209/26/1348659242_7055.jpg)

* 使用直接指针访问：那么Java堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而reference中存储的直接就是对象地址。

![](http://img.my.csdn.net/uploads/201209/26/1348658605_5211.jpg)\)

* 使用句柄来访问的最大好处就是reference中存储的是稳定的句柄地址，在对象被移动（垃圾回收）时只改变句柄中的实例数据指针，而reference本身不需要修改。
* 使用直接指针访问最大的好处就是速度更快，它节省了一次指针定位的时间开销。虚拟机Sun HotSpot使用第二种方式进行对象访问。

## JVM 对象是否存活判断

#### 引用计数法 \(Reference Counting\)

* 给对象中添加一个引用计数器，每当有一个地方引用它时，计数器值加1； 当引用失效时，计数器减1； 任何时刻计数器为0的对象就是不可能被使用的。
* 引用计数法的缺陷

  ```java
    objA.instance = objB;
    objB.instance = objA;
  ```

  * 实际上两个对象已经不可能再被访问，但是它们相互引用着对方，导致他们的引用计数都不为0。于是引用计数器无法通知GC收集器回收它们。

#### 可达性分析算法（Reachability Analysis）

* 算法的基本思路是通过一些被称为“GC Root”的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链\( Reference Chain\)， 当一个对象与 GC Roots 没有任何引用链相连时，则证明此对象是不可用的。
* 在Java语言中，可作为GC Roots的对象包括下面几种：
  * 虚拟机栈（栈帧中的本地变量表）中引用的对象。
  * 方法区中类静态属性引用的对象。
  * 方法区中常量引用的对象。
  * 本地方法栈中JNI引用的对象。

#### 引用的扩充

* JDK1.2之前，Java中的引用的定义很传统：如果reference类型的数据中存储的数值代表的是另一块内存的起始地址，就称这块内存代表着一个引用。这种定义很纯粹，但是太过狭隘，一个对象在这种定义下只有被引用或者没有被引用两种状态。我们还希望能描述这样一类对象：当内存空间还足够时，则保留在内存中；如果内存空间在进行垃圾收集后还是非常紧张，则可以抛弃这些对象。
* JDK1.2之后，Java对引用的概念进行了扩充，将引用分为：
  * 强引用（Strong Reference）： 类似Object obj = new Object\(\); 只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象。
  * 软引用（Soft Reference）：用来描述一些还有用但并非必须的对象。
  * 弱引用（Week Reference）：被软引用引用的对象只能生存到下一次垃圾收集发生之前。
  * 虚引用（Phanotom Reference）：最弱的引用关系，一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来获取一个对象的实例。为一个对象设置虚引用关联的唯一目的是能在这个对象被垃圾收集器回收时收到一个系统通知。
* 即使在可达性分析算法中不可达的对象，也并非是“非死不可”的，要宣告对象的死亡。至少要经历两次标记的过程。
  * 如果对象在进行可达性分析后发现没有与GC Roots相连接的引用链，那它将会被第一次标记并且进行一次筛选，筛选的条件是对象是否有必要执行finalize\(\)方法。当对象没有覆盖finalize\(\)方法，或者finalize\(\)方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要执行”。
  * 如果这个对象被判定有必要执行finalize\(\)方法，那么这个对象将会放置在一个叫做F-Queue的队列之中，并且稍后由一个虚拟机自动建立的、低优先级的Finalizer线程去执行它。这里所谓的“执行”是指虚拟机会触发这个方法，但并不承诺等待它结束运行，这样做的原因是，如果一个对象在finalize\(\)方法中执行缓慢，或者发生了死循环，将很可能导致F-Queue队列中其他对象永远处于等待，甚至导致整个内存回收系统崩溃。finalize\(\)方法是对象逃脱死亡命运的最后一次机会，稍后GC将对F-Queue中的对象进行第二次小规模的标记，如果对象要在finalize\(\)中拯救自己--只要重新与引用链上的任何一个对象建立关联即可，譬如把自己\(this\)赋值给某个变量或者对象的成员变量，那在第二次标记时它将被移除“即将回收”的集合；如果对象这时候还没有逃脱，那基本上它就真的被回收了。

### 回收方法区 \( 永久代）

永久代的回收效率特别低。

永久代的垃圾收集主要回收两部分内容： 废弃常量 和 无用的类。

* 废弃常量：假如一个字符串“abc” 已经进入了常量池，但是当前系统没有任何一个String对象引用了了常量池中的“abc”常量，也没有其他地方引用了这个字面量，如果这时候发生内存回收，而且有必要的话，这个"abc"常量将会被系统清理出常量池。
* 判断无用类的三个条件：
  * 该类的所有实例都已经被回收，也就是Java堆中不存在该类的任何实例。
  * 加载该类的ClassLoader已经被回收。
  * 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

