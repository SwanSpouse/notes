# 2PC和3PC区别

## 3PC为什么比2PC好？

直接分析协调者和参与者都挂的情况。

第二阶段协调者和参与者挂了，挂了的这个参与者在挂之前已经执行了操作。但是由于他挂了，没有人知道他执行了什么操作。

* 这种情况下，当新的协调者被选出来之后，他同样是询问所有的参与者的情况来觉得是commit还是roolback。这看上去和二阶段提交一样啊？他是怎么解决一致性问题的呢？
* 看上去和二阶段提交的那种数据不一致的情况的现象是一样的，但仔细分析所有参与者的状态的话就会发现其实并不一样。我们假设挂掉的那台参与者执行的操作是commit。那么其他没挂的操作者的状态应该是什么？他们的状态要么是prepare-commit要么是commit。因为3PC的第三阶段一旦有机器执行了commit，那必然第一阶段大家都是同意commit。所以，这时，新选举出来的协调者一旦发现未挂掉的参与者中有人处于commit状态或者是prepare-commit的话，那就执行commit操作。否则就执行rollback操作。这样挂掉的参与者恢复之后就能和其他机器保持数据一致性了。（为了简单的让大家理解，笔者这里简化了新选举出来的协调者执行操作的具体细节，真实情况比我描述的要复杂）

**简单概括一下就是，如果挂掉的那台机器已经执行了commit，那么协调者可以从所有未挂掉的参与者的状态中分析出来，并执行commit。如果挂掉的那个参与者执行了rollback，那么协调者和其他的参与者执行的肯定也是rollback操作。**

所以，再多引入一个阶段之后，3PC解决了2PC中存在的那种由于协调者和参与者同时挂掉有可能导致的数据一致性问题。

## 3PC好于2PC的例子

这里举一个两阶段造成数据不一致的例子：比如说有cordinator、cohort1、cohort2三个参与者

首先，请求阶段，cordinator向cohort1和cohort2 发送请求询问是否有事务提交，cohort1和cohort2成功发送请求，有事务需要进行提交。

然后，提交阶段，cordinator向cohort1和cohort2 发送commit命令。此时cordinator宕机，cohort1收到commit命令，成功提交事务之后宕机，cohort2因网络原因没有收到commit请求。此时，cordinator1恢复，向所有节点查询事务提交状态，cohort2回复事务未提交。cordinator、cohort达成一致，同步状态后，继续接受请求。

处理并接受了事务2、3、4等一些列请求。此时 cohort1恢复，和cordinator同步状态之后发现数据不一致，且无法进行恢复。数据不一致。

3PC好就好在，当cordinator进行恢复的时候，如果有一个cohort的状态是preCommit或者commit，那么就执行commit操作；否则abort。

## 3PC存在的问题

在doCommit阶段，如果参与者无法及时接收到来自协调者的doCommit或者rebort请求时，会在等待超时之后，会继续进行事务的提交。

所以，由于网络原因，协调者发送的abort响应没有及时被参与者接收到，那么参与者在等待超时之后执行了commit操作。这样就和其他接到abort命令并执行回滚的参与者之间存在数据不一致的情况。

## reference

* [https://blog.csdn.net/yyd19921214/article/details/68953629](https://blog.csdn.net/yyd19921214/article/details/68953629)

