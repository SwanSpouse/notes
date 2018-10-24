### 分布式系统的Raft算法

过去, Paxos一直是分布式协议的标准，但是Paxos难于理解，更难以实现，Google的分布式锁系统Chubby作为Paxos实现曾经遭遇到很多坑。

来自Stanford的新的分布式协议研究称为Raft，它是一个为真实世界应用建立的协议，主要注重协议的落地性和可理解性。

在Raft系统中，任何时候一个服务器可以扮演下面的角色之一：

* 领导者 Leader：处理所有客户端交互、日志复制等动作，一般一次只有一个领导者。
* 选民Follower：类似选民，完全被动的角色，这样的服务器等待被通知投票。
* 候选人Candidate：候选人就是在选举过程中提名自己的实体，一旦选举成功，则成为领导者。

Raft算法分为两个阶段：首先是选举过程，然后在选举出来的领导人带领下进行正常的操作，比如日志复制等。

* 1.任何一个服务器都可以成为一个Candidate，它向其他的服务器（Follower）发出要求选举自己的请求。
* 2.其他服务器同意了，回复OK（同意）指令

注意如果在这个过程中，有一个Follower当机，没有收到请求选举的要求，因此候选者可以自己选自己，只要达到N/2 + 1 的大多数票，候选人还是可以成为Leader的。

* 3.这样这个Candidate就成为Leader，它可以向Follower发出要执行具体操作动作的指令，比如进行日志复制。

* 4.如果一旦这个Leader宕机崩溃了，那么Follower中会有一个成为Candidate，发出邀票选举，相当于再次执行1.2步骤。

值得注意的是，整个选举过程是有一个时间限制的，如下图：

![](/assets/raft协议.png)

Splite Vote是因为如果同时有两个Candidate向大家邀票，这时通过类似加时赛来解决，两个Candidate在一段timeout比如300ms互相不服气的等待以后，因为双方得到的票数是一样的，一半对一半，那么在300ms以后，再由这两个Candidate发出邀票，这时同时的概率大大降低，那么首先发出邀票的的Candidate得到了大多数同意，成为领导者Leader，而另外一个Candidate后来发出邀票时，那些Follower选民已经投票给第一个Candidate，不能再投票给它，它就成为落选者了，最后这个落选者也成为普通Follower一员了。

#### 日志复制

下面以日志复制为例子说明Raft算法：

* 假设Leader领导人已经选出，这时客户端发出增加一个日志的要求，比如日志是"sally"：

* Leader要求Followe遵从他的指令，都将这个新的日志内容追加到他们各自日志中。

* 大多数follower服务器将日志写入磁盘文件后，确认追加成功，发出Commited Ok。

* 在下一个心跳heartbeat中，Leader会通知所有Follwer更新commited 项目。

对于每个新的日志记录，重复上述过程。

如果在这一过程中，发生了网络分区或者网络通信故障，使得Leader不能访问大多数Follwers了，那么Leader只能正常更新它能访问的那些Follower服务器，而大多数的服务器Follower因为没有了Leader，他们重新选举一个Candidate作为Leader，然后这个Leader作为代表于外界打交道，如果外界要求其添加新的日志，这个新的Leader就按上述步骤通知大多数Followers，如果这时网络故障修复了，那么原先的Leader就变成Follower，在失联阶段这个老Leader的任何更新都不能算commit，都回滚，接受新的Leader的新的更新。

#### redis选举领头sentinel

当一个主服务器被判断为客观下线的时，监视这个下线主服务器的各个sentinel会进行协商。选举一个领头sentinel，并由领头sentinel对下线服务器执行故障转移操作。

* 所有在线的Sentinel都有被选为领头sentinel的资格，换句话说，监视同一个主服务器的多个在线Sentinel中的任意一个都有可能成为领头Sentinel。

* 每次进行领头Sentinel选举之后，不论选举是否成功，所有sentinel的配置纪元（configuration epoch）的值都会自增一次。配置纪元实际上就是一个计数器，并没有什么特别的。

* 在一个配置纪元里面，所有Sentinel都有一次将某个Sentinel设置为局部领头Sentinel的机会，并且局部领头Sentinel一旦设置，在这个纪元里面就不能再更改。

* 每个发现主服务器进入客观下线的Sentinel都会要求其他Sentinel将自己设置为局部领头Sentinel。

* 当一个Sentinel（源Sentinel）向另一个Sentinel（目标Sentinel）发送SENTINEL is-master-down-by-addr命令，并且命令中的runid参数不是\*符号而是源Sentinel的运行ID时，表示源Sentinel要求目标Sentinel将前者设置为后者的局部领头Sentinel。

* Sentinel设置局部领头Sentinel的规则是先到先得：最先向目标Sentinel发送设置要求的源Sentinel将成为目标Sentinel的局部领头Sentinel，而之后接受到所有设置要求都会被目标Sentinel拒绝。

* 目标Sentinel在接受到SENTINEL is-master-down-by-addr命令之后，将向源Sentinel返回一条命令回复，回复中的leader\_runid参数和leader\_epoch参数分别记录了目标Sentinel的局部领头Sentinel的运行ID和配置纪元。

* 源Sentinel在接收到目标Sentinel返回的命令之后，会检查回复中的leader\_epoch参数的值和自己的配置纪元是否相同，如果相同的话，那么源Sentinel继续取出回复中的leader\_runid参数，如果leader\_runid参数的值和源Sentinel的运行ID一致，那么表示目标Sentinel将源Sentinel设置成了局部领头Sentinel。

* 如果有某个Sentinel被半数以上的Sentinel设置成了局部领头Sentinel，那么这个Sentinel成为领头Sentinel。在一个由10个Sentinel组成的Sentinel系统里面，只要有大于等于10/2+1=6个Sentinel将某个Sentinel设置为局部领头Sentinel，那么被设置的那个Sentinel就会成为领头Sentinel。

* 因为领头Sentinel的产生需要半数以上的Sentinel支持，并且每个Sentinel在每个配置纪元里面只能设置一次局部领头Sentinel，所以在一个配置纪元里面，只会出现一个领头Sentinel。

* 如果在给定时限内，没有一个Sentinel被选举为领头Sentinel，那么各个Sentinel将在一段时间之后再次进行选举，直到选举出领头Sentinel为止。

#### reference

* [https://www.jdon.com/artichect/raft.html](https://www.jdon.com/artichect/raft.html)
* [https://raft.github.io/](https://raft.github.io/)
* [https://www.jdon.com/artichect/raft.html](https://www.jdon.com/artichect/raft.html)



