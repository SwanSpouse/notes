### channel

Go语言提倡“应该以通信作为手段来共享内存”。其中最直接、最重要的体现就是channel。

Go语言的channel就像是一个类型安全的通用型管道。Channel提供了一种机制。它既可以同步两个被并发执行的函数，又可以让这两个函数通过相同传递特定类型的值来进行通信。虽然在有些时候使用共享变量和传统的同步方法也可以实现上述用途，但是，作为一个更高级别的方法，使用channel可以使我们更加容易地编写清晰、正确的程序。



#### 项目中使用channel实现函数超时机制


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
	return errors.New("Fail")
}

```
