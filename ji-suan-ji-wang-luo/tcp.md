---
title: TCP三次握手、四次挥手
date: '2018-09-09T19:18:31.000Z'
tags:
  - tcp
  - network
---

# TCP

## TCP状态转换图

![TCP&#x94FE;&#x63A5;&#x72B6;&#x6001;&#x56FE;](https://s1.ax1x.com/2018/09/09/iiwHMT.jpg)

## TCP三次握手

![TCP&#x4E09;&#x6B21;&#x63E1;&#x624B;](https://s1.ax1x.com/2018/09/09/iiwgsS.png)

### TCP 三次握手流程

最初两端的TCP进程都处于CLOSED关闭状态，A主动打开连接，而B被动打开连接。（A、B关闭状态CLOSED——B收听状态LISTEN——A同步已发送状态SYN-SENT——B同步收到状态SYN-RCVD——A、B连接已建立状态ESTABLISHED）

* 1）第一次握手：A的TCP客户进程也是首先创建传输控制块TCB，然后向B发出连接请求报文段，（首部的同步位SYN=1，初始序号seq=x），（SYN=1的报文段不能携带数据）但要消耗掉一个序号，此时TCP客户进程进入SYN-SENT（同步已发送）状态。
* 2）第二次握手：B收到连接请求报文段后，如同意建立连接，则向A发送确认，在确认报文段中（SYN=1，ACK=1，确认号ack=x+1，初始序号seq=y），测试TCP服务器进程进入SYN-RCVD（同步收到）状态；
* 3）第三次握手：TCP客户进程收到B的确认后，要向B给出确认报文段（ACK=1，确认号ack=y+1，序号seq=x+1）（初始为seq=x，第二个报文段所以要+1），ACK报文段可以携带数据，不携带数据则不消耗序号。TCP连接已经建立，A进入ESTABLISHED（已建立连接）。

  当B收到A的确认后，也进入ESTABLISHED状态。

ps: TCB传输控制块Transmission Control Block，存储每一个连接中的重要信息，如TCP连接表，到发送和接收缓存的指针，到重传队列的指针，当前的发送和接收序号。

### 两次握手可以吗

主要为了防止已失效的连接请求报文段突然又传送到了B，因而产生错误。如A发出连接请求，但因连接请求报文丢失而未收到确认，于是A再重传一次连接请求。后来收到了确认，建立了连接。数据传输完毕后，就释放了连接，A工发出了两个连接请求报文段，其中第一个丢失，第二个到达了B，但是第一个丢失的报文段只是在某些网络结点长时间滞留了，延误到连接释放以后的某个时间才到达B，此时B误认为A又发出一次新的连接请求，于是就向A发出确认报文段，同意建立连接，不采用三次握手，只要B发出确认，就建立新的连接了，此时A不理睬B的确认且不发送数据，则B一致等待A发送数据，浪费资源。

前两次握手的作用包括同步消息序列号，如果两次握手可以，那么B在发送ack=x+1之后就可以向A发送消息了。假如B在回复ack=x+1之后，向A连续发送三条数据seq分别为y+1 y+2 y+3，但是如果B发送的ack=x+1丢失的话，A没有同步到B的序列号，A就无法得知B向A发送第一条消息的序列号是多少。所以三次握手很重要的一个信息是同步序列号，

* A: 我的序列号从x开始，
* B: 收到，你的序列号从x开始；我的序列号从y开始
* A: 收到，你的序列号从y开始。

### Server端SYN攻击

服务器端的资源分配是在二次握手时分配的，而客户端的资源是在完成三次握手时分配的，所以服务器容易受到SYN洪泛攻击，SYN攻击就是Client在短时间内伪造大量不存在的IP地址，并向Server不断地发送SYN包，Server则回复确认包，并等待Client确认，由于源地址不存在，因此Server需要不断重发直至超时，这些伪造的SYN包将长时间占用未连接队列，导致正常的SYN请求因为队列满而被丢弃，从而引起网络拥塞甚至系统瘫痪。

防范SYN攻击措施：降低主机的等待时间使主机尽快的释放半连接的占用，短时间受到某IP的重复SYN则丢弃后续请求。

## TCP四次挥手

![](https://s1.ax1x.com/2018/09/09/ii0My8.png)

假设A发起中断请求（FIN报文），意思是说，我（A\)已经没有数据要发送给你了，但你如果还有数据没有发送完成，可以继续发送。B收到A的FIN请求后，返回ACK告诉A，你的请求我收到了，但是我还没有准备好，请继续等待我的消息。收到B回复的ACK，A就进入了FIN\_WAIT状态，接受B发送的数据，同时等待B的FIN报文。当B已经确定数据发送完成，则向A发送FIN报文，告诉A，我已经没有数据发送给你了，准备好关闭连接了。A收到B的FIN报文后，就知道可以关闭连接了，但是A还是不相信网络，担心B不知道它已经收到消息，所以发送ACK，发送后进入TIME\_WAIT状态，如果B没有收到ACK则可以重传。B收到ACK后，就知道可以断开连接了。A在等待了2ms后依然没有回应，则表明B已经正常关闭，A再关闭连接。

* 1）A的应用进程先向其TCP发出连接释放报文段（FIN=1，序号seq=u），并停止再发送数据，主动关闭TCP连接，进入FIN-WAIT-1（终止等待1）状态，等待B的确认。
* 2）B收到连接释放报文段后即发出确认报文段，（ACK=1，确认号ack=u+1，序号seq=v），B进入CLOSE-WAIT（关闭等待）状态，此时的TCP处于半关闭状态，A到B的连接释放。
* 3）A收到B的确认后，进入FIN-WAIT-2（终止等待2）状态，等待B发出的连接释放报文段。
* 4）B没有要向A发出的数据，B发出连接释放报文段（FIN=1，ACK=1，序号seq=w，确认号ack=u+1），B进入LAST-ACK（最后确认）状态，等待A的确认。
* 5）A收到B的连接释放报文段后，对此发出确认报文段（ACK=1，seq=u+1，ack=w+1），A进入TIME-WAIT（时间等待）状态。此时TCP未释放掉，需要经过时间等待计时器设置的时间2MSL后，A才进入CLOSED状态。

### 为什么连接的时候是三次握手，关闭的时候却是四次握手？

因为当Server端收到Client端的SYN连接请求报文后，可以直接发送SYN+ACK报文。其中ACK报文是用来应答的，SYN报文是用来同步的。但是关闭连接时，当Server端收到FIN报文时，很可能并不会立即关闭SOCKET，所以只能先回复一个ACK报文，告诉Client端，"你发的FIN报文我收到了"。只有等到我Server端所有的报文都发送完了，我才能发送FIN报文，因此不能一起发送。故需要四步握手。

## reference

* [https://www.cnblogs.com/Andya/p/7272462.html](https://www.cnblogs.com/Andya/p/7272462.html)

