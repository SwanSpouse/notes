# /dev/null

标准输入0 从键盘获得输入 /proc/self/fd/0

标准输出1 输出到屏幕（即控制台） /proc/self/fd/1

错误输出2 输出到屏幕（即控制台） /proc/self/fd/2

/dev/null代表linux的空设备文件，所有往这个文件里面写入的内容都会丢失，俗称“黑洞”

1、2&gt;/dev/null意思就是把错误输出到“黑洞”

2、&gt;/dev/null 2&gt;&1默认情况是1，也就是等同于1&gt;/dev/null 2&gt;&1。意思就是把标准输出重定向到“黑洞”，还把错误输出2重定向到标准输出1，也就是标准输出和错误输出都进了“黑洞”

3、2&gt;&1 &gt;/dev/null意思就是把错误输出2重定向到标准出书1，也就是屏幕，标准输出进了“黑洞”，也就是标准输出进了黑洞，错误输出打印到屏幕

关于这里”&”的作用，我们可以这么理解2&gt;/dev/null重定向到文件，那么2&gt;&1，这里如果去掉了&就是把错误输出给了文件1了，用了&是表明1是标准输出。

## reference

* [https://blog.csdn.net/zhongqi2513/article/details/78613768](https://blog.csdn.net/zhongqi2513/article/details/78613768)

