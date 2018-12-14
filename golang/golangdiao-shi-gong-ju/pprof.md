### PProf工具

**go tool pprof http://xxxx/debug/pprof/debug 参数**

  -inuse\_space      Display in-use memory size

  -inuse\_objects    Display in-use object counts

  -alloc\_space      Display allocated memory size

  -alloc\_objects    Display allocated object counts

**各个指标的说明：**

flat、flat% 表示函数在 CPU 上运行的时间以及百分比

sum% 表示当前函数累加使用 CPU 的比例

cum、cum%表示该函数以及子函数运行所占用的时间和比例，应该大于等于前两列的值

#### reference

* [http://fuxiaohei.me/2015/10/14/pugo-mem-leak-profile.html](http://fuxiaohei.me/2015/10/14/pugo-mem-leak-profile.html)



