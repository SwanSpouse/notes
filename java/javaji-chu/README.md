# Java基础

## **volatile**

**volatile的本意是“易变的”。**volatile关键字修饰符是一种类型编译器，用它声明的类型变量表示可以被某些未知的因素更改，比如：操作系统、硬件或者其它线程等。**遇到这个关键字声明的变量，编译器对访问该变量的代码就不再进行优化，从而可以提供对特殊地址的稳定访问。**当要求使用volatile 声明的变量的值的时候，系统总是重新从它所在的内存读取数据，即使它前面的指令刚刚从该处读取过数据。而且读取的数据立刻被保存。volatile 指出 i是随时可能发生变化的，每次使用它的时候必须从i的地址中读取。对于volatile类型的变量，系统每次用到他的时候都是直接从对应的内存当中提取，而不会利用cache当中的原有数值，以适应它的未知何时会发生的变化。

## fail-fast

**fail-fast 机制是java集合\(Collection\)中的一种错误机制。**当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。但java JDK并不保证fail-fast一定会发生。

## fail-fast实现

```java
/**
 * The number of times this list has been <i>structurally modified</i>.
 * Structural modifications are those that change the size of the
 * list, or otherwise perturb it in such a fashion that iterations in
 * progress may yield incorrect results.
 *
 * <p>This field is used by the iterator and list iterator implementation
 * returned by the {@code iterator} and {@code listIterator} methods.
 * If the value of this field changes unexpectedly, the iterator (or list
 * iterator) will throw a {@code ConcurrentModificationException} in
 * response to the {@code next}, {@code remove}, {@code previous},
 * {@code set} or {@code add} operations.  This provides
 * <i>fail-fast</i> behavior, rather than non-deterministic behavior in
 * the face of concurrent modification during iteration.
 *
 * <p><b>Use of this field by subclasses is optional.</b> If a subclass
 * wishes to provide fail-fast iterators (and list iterators), then it
 * merely has to increment this field in its {@code add(int, E)} and
 * {@code remove(int)} methods (and any other methods that it overrides
 * that result in structural modifications to the list).  A single call to
 * {@code add(int, E)} or {@code remove(int)} must add no more than
 * one to this field, or the iterators (and list iterators) will throw
 * bogus {@code ConcurrentModificationExceptions}.  If an implementation
 * does not wish to provide fail-fast iterators, this field may be
 * ignored.
 */
protected transient int modCount = 0;
```

从上面的注释中可以知道，modCount的值会在当底层数组内容被修改的时候，如进行add, remove, clear等操作的时候被改变。

在每次新建Iterator对象的时候，都会用上面的modCount初始化Iterator中的expectedModCount值。并在进行next、remove等操作的时候利用checkForComodification\(\)对 expectedModCount 是否和 modCounts 是否相等进行检查。如果不相等，则抛出ConcurrentModificationException 异常。

## fail-fast解决办法

由于java JDK不保证fail-fast机制一定会发生，因此在多线程情况下解决fail-fast的办法一般有两种:

* 使用java.util.concurrent包下的类去替代java.util包下的类
* 加锁

## 疑问

```java
ArrayList<Integer> list = new ArrayList<>();
Iterator<Integer> it = list.iterator();
list.add(1);

it.next();
```

其实不用在多线程的情况下，上面的代码就可以抛出ConcurrentModificationException。（这种情况下ConcurrentModificationException这个命名是不是有点儿名不符实呐）

在我看来，fail-fast更像是一个强有力的warning。强制地提示你，你的代码里存在隐患。由于多线程的问题不容易被定位，所以编译器强制你使用更加安全的方式来实现你的逻辑。

就像上面的代码，我其实已经知道我正在使用iterator的低层数组被某些方法改变，我只是不想再new一个新的出来，但是仍然会抛出异常来强制你使用更安全的方式来进行操作。

## reference

* java 1.7 source code
* [https://www.cnblogs.com/ccgjava/p/6347425.html?utm\_source=itdadao&utm\_medium=referral](https://www.cnblogs.com/ccgjava/p/6347425.html?utm_source=itdadao&utm_medium=referral) 

