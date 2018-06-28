### Producer consumer pattern

```golang
/*
These next two lines create two new channels, which are 
one of the concurrency constructs golang offers.
The first channel is a boolean channel the second one is
an int channel. We can read or write data to the channels.
We will see how shortly.
*/
var done = make(chan bool)
var msgs = make(chan int)

/*
In our main function, we spawn two go routines. go routines
are cheap functions that will be run concurrently by go runtime. 
They are just regular functions, but using the keyword go before
a function call we can run them as go routines.
So those two functions will run concurrently.
Finally we block the main thread by reading from the done channel.
As soon something comes down the channel, we read it, toss it and
the main thread continues executing. In this case we exit the 
program.
*/
func main () {
   go produce()
   go consume()
   <- done
}

/*
The produce go routine (or function) loops 10 times
and writes an integer (0..10) in the msg channel.
Notice that we will block until someone (the consumer)
reads form the other side of the channel. 
Once we are done, we send a boolean on the done channel
to let the main go routine that we are done.
*/
func produce() {
    for i := 0; i < 10; i++ {
        msgs <- i
    }
    done <- true
}

/*
The consume go routine loops infinitely and reads 
on the msgs channel. It will block until something 
comes in the channel. 
The syntax can be a little bit strange for people 
coming from other languages.
':=' creates a variable assigning it the type of the value
coming on the right of the assignation. An int in this 
case.
'<-' is the go way to read from a channel.
Once we have the msg (int) we dump it in the stdout.
*/
func consume() {
    for {
      msg := <-msgs
      fmt.Println(msg)
   }
}
```



#### reference 

* https://gist.github.com/drio/dd2c4ad72452e3c35e7e



