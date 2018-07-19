### golang 逃逸分析

#### 逃逸分析：

逃逸分析是一种确定指针动态范围的方法，可以分析在程序的哪些地方可以访问到指针。它涉及到指针分析和形状分析。 当一个变量\(或对象\)在子程序中被分配时，一个指向变量的指针可能逃逸到其它执行线程中，或者去调用子程序。如果使用尾递归优化（通常在函数编程语言中是需要的），对象也可能逃逸到被调用的子程序中。 如果一个子程序分配一个对象并返回一个该对象的指针，该对象可能在程序中的任何一个地方被访问到——这样指针就成功“逃逸”了。如果指针存储在全局变量或者其它数据结构中，它们也可能发生逃逸，这种情况是当前程序中的指针逃逸。 逃逸分析需要确定指针所有可以存储的地方，保证指针的生命周期只在当前进程或线程中。

#### 逃逸分析的作用

* 最大的好处应该是减少gc的压力，不逃逸的对象分配在栈上，当函数返回时就回收了资源，不需要gc标记清除。

* 因为逃逸分析完后可以确定哪些变量可以分配在栈上，栈的分配比堆快，性能好。

* 同步消除，如果你定义的对象的方法上有同步锁，但在运行时，却只有一个线程在访问，此时逃逸分析后的机器码，会去掉同步锁运行。

go在一定程度消除了堆和栈的区别，因为go在编译的时候进行逃逸分析，来决定一个对象放栈上还是放堆上，不逃逸的对象放栈上，可能逃逸的放堆上。

##### How do I know whether a variable is allocated on the heap or the stack? {#How-do-I-know-whether-a-variable-is-allocated-on-the-heap-or-the-stack}

From a correctness standpoint, you don’t need to know. Each variable in Go exists as long as there are references to it. The storage location chosen by the implementation is irrelevant to the semantics of the language.

准确地说，你并不需要知道。Golang 中的变量只要被引用就一直会存活，存储在堆上还是栈上由内部实现决定而和具体的语法没有关系。

The storage location does have an effect on writing efficient programs. When possible, the Go compilers will allocate variables that are local to a function in that function’s stack frame. However, if the compiler cannot prove that the variable is not referenced after the function returns, then the compiler must allocate the variable on the garbage-collected heap to avoid dangling pointer errors. Also, if a local variable is very large, it might make more sense to store it on the heap rather than the stack.

知道变量的存储位置确实和效率编程有关系。如果可能，Golang 编译器会将函数的局部变量分配到函数栈帧（stack frame）上。然而，如果编译器不能确保变量在函数 return 之后不再被引用，编译器就会将变量分配到堆上。而且，如果一个局部变量非常大，那么它也应该被分配到堆上而不是栈上。

In the current compilers, if a variable has its address taken, that variable is a candidate for allocation on the heap. However, a basic_escape analysis_recognizes some cases when such variables will not live past the return from the function and can reside on the stack.

当前情况下，如果一个变量被取地址，那么它就有可能被分配到堆上。然而，还要对这些变量做逃逸分析，如果函数 return 之后，变量不再被引用，则将其分配到栈上。



#### 参考

* [https://studygolang.com/articles/10026](https://studygolang.com/articles/10026)

* Go 逃逸分析的缺陷: [https://studygolang.com/articles/12396?fr=sidebar](https://studygolang.com/articles/12396?fr=sidebar)

* [https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-stacks-and-pointers.html](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-stacks-and-pointers.html)

* http://legendtkl.com/2017/04/02/golang-alloc/



