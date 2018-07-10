### 调度器追踪

使用GODEBUG跟踪器

在执行程序前先设置好环境变量，如果需要更详细的调度器信息可以添加参数scheddetail

```go
GOMAXPROCS=1 GODEBUG=schedtrace=1000 ./test
GOMAXPROCS=1 GODEBUG=schedtrace=1000,scheddetail=1 ./test
```

#### 调度器摘要信息

```shell
SCHED 3009ms: gomaxprocs=1 idleprocs=0 threads=3 spinningthreads=0 idlethreads=1 runqueue=0 [9]
```

| 输出项 | 意义 |
| :--- | :--- |
| 3009ms | 自从程序开始的毫秒数 |
| gomaxprocs=1 | 配置的处理器数\(逻辑的processor，也就是Go模型中的P,会通过操作系统的线程绑定到一个物理处理器上\) |
| idleprocs=0 | 空闲的处理器数,当前0个空闲 |
| threads=3 | 运行期管理的线程数，目前三个线程 |
| idlethreads=1 | 空闲的线程数,当前一个线程空闲，两个忙 |
| runqueue=0 | 在全局的run队列中的goroutine数，目前所有的goroutine都被移动到本地run队列。 |
| \[9\] | 本地run队列中的goroutine数，目前9个goroutine在本地run队列中等待。 |

#### ![](/assets/godebug sched.png)

#### 

#### reference

* http://colobu.com/2016/04/19/Scheduler-Tracing-In-Go/



