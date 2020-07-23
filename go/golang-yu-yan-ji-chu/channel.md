# Channel

## channel

Go语言提倡“应该以通信作为手段来共享内存”。其中最直接、最重要的体现就是channel。

Go语言的channel就像是一个类型安全的通用型管道。Channel提供了一种机制。它既可以同步两个被并发执行的函数，又可以让这两个函数通过相同传递特定类型的值来进行通信。虽然在有些时候使用共享变量和传统的同步方法也可以实现上述用途，但是，作为一个更高级别的方法，使用channel可以使我们更加容易地编写清晰、正确的程序。

## channel的常见用法

### 超时机制

首先声明一个存放error的statusChan channel。如果在15秒之内，checkXXXXStatus能将执行结果写入到statusChan中，那么就直接获取checkXXXXStatus的执行结果；但如果没有在15秒之内将结果写入statusChan之内，那么case2中的计时器将触发。返回执行超时。

```go
statusChan := make(chan error, 1)
go checkXXXXStatus(statusChan)

select {
case err := <-statusChan:
    myErr, ok := err.(*MyErr)
    if ok {
        if myErr.Code() == qe.OK {
            return nil
        }
    }
    return err
case <-time.After(time.Second * 15):
    return errors.New("timeout") 
}
```

### 退出机制

首先声明一个exitChan，作为参数传入goroutine中。当这个exitChan被关闭或者收到消息的时候，则表示当前goroutine应该停止运行。如下述代码，当exitChan被关闭的时候，都会触发&lt;-exitChan这个case，然后退出当前goroutine的运行。当然这个功能也可以通过waitGroup的方式来进行实现。

```go
func thread(exitChan chan int, num int) {
    for {
        select {
        case <-exitChan:
            fmt.Println("current thread exit ", num)
            return 
        default:
            fmt.Println("current thread in a loop ", num)
            time.Sleep(time.Millisecond)
        }
    }
}

func main() {
    exitChan := make(chan int)
    go thread(exitChan, 1)
    go thread(exitChan, 2)

    time.Sleep(2 * time.Millisecond)

    close(exitChan)
}
```

### 判断当前channel是否被阻塞

在select子句中添加default，当default被执行的是时候。说明当前的msgChan没有消息。可以进行一些处理。防止一直goroutine一直阻塞在&lt;-msgChan这里。

```go
func handleMessage(msgChan chan int, num int) {
    for {
        select {
        case <-msgChan:
            fmt.Println("current channel receive a message", num)
        default:
            fmt.Println("current channel is blocked ", num)
            time.Sleep(time.Millisecond)
        }
    }
}
```

