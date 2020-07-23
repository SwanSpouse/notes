# Golang 并发处理demo

下面代码中需要说明的几个点：

* Start, Close, AddTask 是不可以并发调用的。
* 通过SafeGo捕获所有可能的panic。其实在发生panic的时候可以调用os.Exit\(1\)。否则如果存在异常数据的时候，虽然启动了10个Gorountine，但是可能有9个因为panic停掉了，最后就变成单线程的了。通过调用os.Exit\(1\)可以停掉整个进程，去解决逻辑中的问题，然后再重新运行嘛。
* 假定Worker的数量是10，在启动的时候，会启动10个worker，所有的worker都尝试从Tasks的channel中获取要执行的任务。在没有调用addTask之前，所有的worker都阻塞在从task channel中获取task。直到调用了addTask。
* 当调用了Close之后，task channel被关闭。程序等待所有的worker消费完task channel中所有的task，程序结束。

```go
type Executor struct {
    Worker  int
    Tasks   chan func()
    wg      *sync.WaitGroup
    started bool
}

func SafeGo(f func(), msg string) {
    defer func() {
        if err := recover(); err != nil {
            msg = fmt.Sprintf("%s \n recover error: %+v \n stacktrace \n %s", msg, err, string(debug.Stack()))
            logger.Error(msg)
        }
    }()
    f()
}

func (e *Executor) Start() {
    if e.started {
        return
    }

    i := 0
    for i < e.Worker 
        i++
        e.wg.Add(1)
        go func(i int) {
            taskNum := 0
            logger.Info("start executor worker %d", i)
            for task := range e.Tasks {
                SafeGo(task, "executor task panic")
                taskNum++
            }
            e.wg.Done()
            logger.Info("end executor worker %d, finish %d tasks", i, taskNum)

        }(i)
    }
    e.started = true
    logger.Info("start executor %d workers", e.Worker)
}

func (e *Executor) Close() {
    if !e.started {
        return
    }

    e.started = false
    close(e.Tasks)
    e.wg.Wait()
    logger.Info("close executor successfully")
}

func (e *Executor) AddTask(f func()) {
    if e.started {
        e.Tasks <- f
    }
}

func NewExecutor(workerNum int, queueSize int) *Executor {
    if workerNum <= 0 {
        workerNum = 1
    }

    if queueSize < workerNum {
        queueSize = 2 * workerNum
    }

    return &Executor{
        Worker:  workerNum,
        Tasks:   make(chan func(), queueSize),
        started: false,
        wg:      &sync.WaitGroup{},
    }
}
```

## reference

* qxf-backend

