# Channel有缓冲区和无缓冲区的区别

## golang channel 有缓冲 与 无缓冲 是有重要区别的

我之前天真的认为 有缓冲与无缓冲的区别 只是 无缓冲的 是 默认 缓冲 为1 的缓冲式

其实是彻底错误的，无缓冲的与有缓冲channel有着重大差别

那就是一个是同步的 一个是非同步的

怎么说？比如

c1:=make\(chan int\) 无缓冲

c2:=make\(chan int,1\) 有缓冲

c1&lt;-1

无缓冲的 不仅仅是 向 c1 通道放 1 而是 一直要有别的携程 &lt;-c1 接手了 这个参数，那么c1&lt;-1才会继续下去，要不然就一直阻塞着

而 c2&lt;-1 则不会阻塞，因为缓冲大小是1 （其实是缓冲大小为0）只有当 放第二个值的时候 第一个还没被人拿走，这时候才会阻塞。

打个比喻

无缓冲的 就是一个送信人去你家门口送信 ，你不在家 他不走，你一定要接下信，他才会走。

无缓冲保证信能到你手上

有缓冲的 就是一个送信人去你家仍到你家的信箱 转身就走 ，除非你的信箱满了 他必须等信箱空下来。

有缓冲的 保证 信能进你家的邮箱

## reference

* [https://my.oschina.net/u/157514/blog/149192](https://my.oschina.net/u/157514/blog/149192)

